# Staking with jormungandr

Here we will describe how to:

* _delegate your stake to a stake pool_ so that you can participate to the consensus
  and maybe collect rewards for that;

If you are a user of the blockchain then you will certainly not need to look
at the last section about _how to register a stake pool_.



# Delegating your stake


## how to create the delegation certificate

Stake is concentrated in accounts, and you will need your account key to
delegate its associated stake.

You will need your:

* account public key: a bech32 string of a public key
* the Stake Pool ID: an hexadecimal string identifying the stake pool you want
  to delegate your stake to.

```
$ jcli certificate new stake-delegation STAKE_POOL_ID ACCOUNT_PUBLIC_KEY > stake_delegation.cert
```

## how to sign your delegation certificate

We need to make sure that the owner of the account is authorizing this
delegation to happens, and for that we need a cryptographic signature.

We will need the account secret key to create a signature

```
$ cat stake_delegation.cert | jcli certificate sign account_key.prv | tee stake_delegation.cert
cert1q8rv4ccl54k99rtnm39...zr0
```

The output can now be added in the `transaction` and submitted to a node.

## submitting to a node

The `jcli transaction add-certificate` command should be used to add a certificate **before finalizing** the transaction.

For example:

```sh

...

jcli transaction add-certificate $(cat stake_delegation.cert) --staging tx

jcli transaction finalize CHANGE_ADDRESS --fee-constant 5 --fee-coefficient 2 --fee-certificate 2 --staging tx

...

```

The `--fee-certificate` flag indicates the cost of adding a certificate, used for computing the fees, it can be omitted if it is zero.

See [here](../jcli/transaction.md) for more documentation on transaction creation.



# registering stake pool

There are multiple components to be aware of when running a stake pool:

* your `NodeId`: it is the identifier within the blockchain protocol (wallet
  will delegate to your stake pool via this `NodeId`);
* your [**VRF**] key pairs: this is the cryptographic material we will use to participate
  to the leader election;
* your [**KES**] key pairs: this is the cryptographic material we will use to sign the
  block with.

So in order to start your stake pool you will need to generate these objects.

# The primitives

## VRF key pair

To generate your [**VRF**] Key pairs, we will utilise [`jcli`] as described
[here](../jcli/key.md):

```
$ jcli key generate --type=Curve25519_2HashDH > stake_pool_vrf.prv
```

`stake_pool_vrf.prv` file now contains the VRF private key.

```
$ cat stake_pool_vrf.prv | jcli key to-public > stake_pool_vrf.pub
```

## KES key pair

Similar to above:

```
$ jcli key generate --type=SumEd25519_12 > stake_pool_kes.prv
```

`stake_pool_kes.prv` now contains your KES private key

```
$ cat stake_pool_kes.prv | jcli key to-public > stake_pool_kes.pub
```

# creating your stake pool certificate

The certificate is what will be sent to the blockchain in order to register
yourself to the other participants of the blockchain that you are a stake
pool too.

```
$ jcli certificate new stake-pool-registration \
    --kes-key $(cat stake_pool_kes.pub) \
    --vrf-key $(cat stake_pool_vrf.pub) \
    --serial 1010101010 > stake_pool.cert
```

Now you need to sign this certificate with the owner key:

```
$ cat stake_pool.cert | jcli certificate sign stake_key.prv | tee stake_pool.cert
cert1qsqqqqqqqqqqqqqqqqqqq0p5avfqp9tzusr26...cegxaz
```

And now you can retrieve your stake pool id (`NodeId`):

```
$ cat stake_pool.cert | jcli certificate get-stake-pool-id | tee stake_pool.id
ea830e5d9647af89a5e9a4d4089e6e855891a533316adf4a42b7bf1372389b74
```

[**VRF**]: https://en.wikipedia.org/wiki/Verifiable_random_function



# Advanced

This section is meant for advanced users and developers of the node, or if
you to learn more about the node.

At the moment, it only covers details on how to create your own blockchain genesis
configuration, but in normal case, the blockchain configuration should be available
with the specific blockchain system.



# genesis file

The genesis file is the file that allows you to create a new blockchain
from block 0. It lays out the different parameter of your blockchain:
the initial utxo, the start time, the slot duration time, etc...

Example of a BFT genesis file with an initial address UTxO and an account UTxO.
More info regarding [starting a BFT blockchain here](./02_starting_bft_blockchain.md)
and [regarding addresses there](../jcli/address.md).
You could also find information regarding the [jcli genesis tooling](../jcli/genesis.md).

You can generate a documented pre-generated genesis file:

```
jcli genesis init
```

For example your genesis file may look like:

```yaml
{{#include ../../jormungandr-lib/src/interfaces/block0_configuration/DOCUMENTED_EXAMPLE.yaml}}
```

There are multiple _parts_ in the genesis file:

* `blockchain_configuration`: this is a list of configuration
  parameters of the blockchain, some of which can be changed later
  via the update protocol;
* `initial`: list of steps to create initial state of ledger

## `blockchain_configuration` options

| option | format | description |
|:-------|:-------|:------------|
| `block0_date` | number | the official start time of the blockchain, in seconds since UNIX EPOCH |
| `discrimination` | string | `production` or `test` |
| `block0_consensus` | string | `bft` |
| `slot_duration` | number | the number of seconds between the creation of 2 blocks |
| `epoch_stability_depth` | number | allowed size of a fork (in number of block) |
| `consensus_leader_ids` | array | the list of the BFT leader at the beginning of the blockchain |
| `max_number_of_transactions_per_block` | number | the maximum number of transactions allowed in a block |
| `bft_slots_ratio` | number | placeholder, do not use |
| `linear_fees` | object | linear fee settings, set the fee for transaction and certificate publishing |
| `consensus_genesis_praos_active_slot_coeff` | number | genesis praos active slot coefficient.  Determines minimum stake required to try becoming slot leader, must be in range (0,1] |
| `kes_update_speed` | number | the speed to update the KES Key in seconds |
| `slots_per_epoch` | number | number of slots in each epoch |

_for more information about the BFT leaders in the genesis file, see
[Starting a BFT Blockchain](./02_starting_bft_blockchain.md)_

## `initial` options

Each entry can be one of 3 variants:

| variant | format | description |
|:-------|:-------|:------------|
| `fund` | sequence | initial deposits present in the blockchain (up to 255 outputs per entry) |
| `cert` | string | initial certificate |
| `legacy_fund` | sequence | same as `fund`, but with legacy Cardano address format |

Example:

```yaml
initial:
  - fund:
      - address: <address>
        value: 10000
      - address: <address2>
        value: 20000
      - address: <address3>
        value: 30000
  - cert: <certificate>
  - legacy_fund:
      - address: <legacy address>
        value: 123
  - fund:
      - address: <another address>
        value: 1001
```

### `fund` and `legacy_fund` format

| variant | format | description |
|:-------|:-------|:------------|
| `address` | string | can be a [single address](../jcli/address.md#address-for-utxo) or an [account address](../jcli/address.md#address-for-account) |
| `value` | number | assigned value |

`legacy_fund` differs only in address format, which is legacy Cardano



# starting a bft node

BFT stands for the Byzantine Fault Tolerant
([read the paper](https://iohk.io/research/papers/#L5IHCV53)).

Jormungandr allows you to start a BFT blockchain fairly easily. The main
downside is that it is centralized, only a handful of nodes will ever have
the right to create blocks.

## How does it work

It is fairly simple. A given number of Nodes (`N`) will generate
a key pairs of type `Ed25519` (see
[JCLI's Keys](./../jcli/key.md)).

They all share the public key and add them in the genesis.yaml file.
It is the source of truth, the file that will generate the first block
of the blockchain: the **Block 0**.

Then, only by one after the other, each Node will be allowed to create a block.
Utilising a Round Robin algorithm.

## Example of genesis file

```yaml
blockchain_configuration:
  block0_date: 1550822014
  discrimination: test
  block0_consensus: bft
  slots_per_epoch: 5
  slot_duration: 15
  epoch_stability_depth: 10
  consensus_leader_ids:
    - ed25519e_pk1k3wjgdcdcn23k6dwr0cyh88ad7a4ayenyxaherfazwy363pyy8wqppn7j3
    - ed25519e_pk13talprd9grgaqzs42mkm0x2xek5wf9mdf0eefdy8a6dk5grka2gstrp3en
  bft_slots_ratio: 0.220
  consensus_genesis_praos_active_slot_coeff: 0.22
  max_number_of_transactions_per_block: 255
  linear_fees:
    constant: 2
    coefficient: 1
    certificate: 4
  kes_update_speed: 43200
initial:
  - fund:
      - address: ta1svy0mwwm7mdwcuj308aapjw6ra4c3e6cygd0f333nvtjzxg8ahdvxlswdf0
        value: 10000
  - cert: cert1qgqqqqqqqqqqqqqqqqqqq0p5avfqqmgurpe7s9k7933q0wj420jl5xqvx8lywcu5jcr7fwqa9qmdn93q4nm7c4fsay3mzeqgq3c0slnut9kns08yn2qn80famup7nvgtfuyszqzqrd4lxlt5ylplfu76p8f6ks0ggprzatp2c8rn6ev3hn9dgr38tzful4h0udlwa0536vyrrug7af9ujmrr869afs0yw9gj5x7z24l8sps3zzcmv
  - legacy_fund:
      - address: 48mDfYyQn21iyEPzCfkATEHTwZBcZJqXhRJezmswfvc6Ne89u1axXsiazmgd7SwT8VbafbVnCvyXhBSMhSkPiCezMkqHC4dmxRahRC86SknFu6JF6hwSg8
        value: 123
```

In order to start your blockchain in BFT mode you need to be sure that:

* `consensus_leader_ids` is non empty;

more information regarding the [genesis file here](./01_the_genesis_block.md).

## Creating the block 0

```
jcli genesis encode --input genesis.yaml --output block-0.bin
```

This command will create (or replace) the **Block 0** of the blockchain
from the given genesis configuration file (`genesis.yaml`).

## Starting the node

Now that the blockchain is initialized, you need to start your node.

Write you private key in a file on your HD:

```
$ cat node_secret.yaml
bft:
  signing_key: ed25519_sk1hpvne...
```

Configure your Node (config.yml) and run the following command:

```
$ jormungandr --genesis-block block-0.bin \
    --config example.config \
    --secret node_secret.yaml
```

It's possible to use the flag `--secret` multiple times to run a node
with multiple leaders.

## Step by step to start the BFT node

1. Generate initial config `jcli genesis init > genesis.yaml`
2. Generate secret key, e.g. ` jcli key generate --type=Ed25519 > key.prv`
3. Put secret key in a file, e.g. `node_secret.yaml` as follows:

```yaml
bft:
 signing_key: ed25519_sk1kppercsk06k03yk4qgea....
```

4. Generate public key out of previously generated key ` cat key.prv |  jcli key to-public`
5. Put generated public key as in `genesis.yaml` under `consensus_leader_ids:`
6. Generate block = `jcli genesis encode --input genesis.yaml --output block-0.bin`
7. Create config file and store it on your HD as `node.config` e.g. ->

```yaml
---
log:
  level: trace
  format: json
rest:
  listen: "127.0.0.1:8607"
p2p:
  public_address: /ip4/127.0.0.1/tcp/8606
  topics_of_interest:
    messages: low
    blocks: normal
```

8. Start Jörmungandr node :
```
jormungandr --genesis-block block-0.bin --config node.config --secret node_secret.yaml
```

# Script

Additionally, there is a script [here](https://github.com/input-output-hk/jormungandr/blob/master/scripts/bootstrap) that can be used to bootstrap a test node with bft consensus protocol.



# starting a genesis blockchain

When starting a genesis praos blockchain there is an element to take
into consideration while constructing the block 0: _the stake distribution_.

In the context of Genesis/Praos the network is fully decentralized and it is
necessary to think ahead about initial stake pools and to make sure there
is stake delegated to these stake pools.

In your genesis yaml file, make sure to set the following values to the appropriate
values/desired values:

```yaml
# The Blockchain Configuration defines the settings of the blockchain.
blockchain_configuration:
  block0_consensus: genesis_praos
  bft_slots_ratio: 0
  consensus_genesis_praos_active_slot_coeff: 0.1
  kes_update_speed: 43200 # 12hours
```

`block0_consensus` set to `genesis_praos` means you want to start a blockchain with
genesis praos as the consensus layer.

`bft_slots_ratio` needs to be set to `0` (we don't support composite modes between
BFT mode and Genesis mode -- yet).

`consensus_genesis_praos_active_slot_coeff` determines minimum stake required to
try becoming slot leader, must be in range 0 exclusive and 1 inclusive.

## The initial certificates

In the `initial_certs` field you will set the initial certificate. This is important
to declare the stake pool and delegate stake to them. Otherwise no block will be ever
created.

Remember that in this array the **order** matters:

In order to delegate your stake, you need a stake pool to already exist, so the stake pool registration certificate should go first.

### Stake pool registration

Now you can register a stake pool.
Follow the instruction in [registering stake pool guide](../stake_pool/registering_stake_pool.md).

The _owner key_ (the key you sign the stake pool registration certificate) is the secret
key associated to a previously registered stake key.

### Delegating stake

Now that there is both your stake key and there are stake pools available
in the block0 you need to delegate to one of the stake pool. Follow the instruction
in [delegating stake](../stake_pool/delegating_stake.md).

And in the initial funds start adding the addresses. To create an address with delegation
follow the instruction in [JCLI's address guide](../jcli/address.md). Utilise the stake key
registered previously as group address:

```
jcli address single $(cat wallet_key.pub) $(cat stake_key.pub)
ta1sjx4j3jwel94g0cgwzq9au7h6m8f5q3qnyh0gfnryl3xan6qnmjse3k2uv062mzj34eacjnxthxqv8fvdcn6f4xhxwa7ms729ak3gsl4qrq2mm
```

You will notice that addresses with delegation are longer (about twice longer) than
address without delegation.

For example, the most minimal setting you may have is:

```yaml
initial_certs:
  # register a stake pool (P), owner of the stake pool is the stake key (K)
  - cert1qsqqqqqqqqqqqqqqqqqqq0p5avfqp9tzusr26chayeddkkmdlap6tl23ceca8unsghc22tap8clhrzslkehdycufa4ywvqvs4u36zctw4ydtg7xagprfgz0vuujh3lgtxgfszqzqj4xk4sxxyg392p5nqz8s7ev5wna7eqz7ycsuas05mrupmdsfk0fqqudanew6c0nckf5tsp0lgnk8e8j0dpnxvjk2usn52vs8umr3qrccegxaz

  # delegate stake associated to stake key (K) to stake pool (P)
  - cert1q0rv4ccl54k99rtnm39xvhwvqcwjcm385n2dwvamahpu5tmdz3plt65rpewev3a03xj7nfx5pz0xap2cjxjnxvt2ma9y9dalzder3xm5qyqyq0lx05ggrws0ghuffqrg7scqzdsd665v4m7087eam5zvw4f26v2tsea3ujrxly243sgqkn42uttk5juvq78ajvfx9ttcmj05lfuwtq9qhdxzr0

initial_funds:
  # address without delegation
  - address: ta1swx4j3jwel94g0cgwzq9au7h6m8f5q3qnyh0gfnryl3xan6qnmjsczt057x
    value: 10000
  # address delegating to stake key (K)
  - address: ta1sjx4j3jwel94g0cgwzq9au7h6m8f5q3qnyh0gfnryl3xan6qnmjse3k2uv062mzj34eacjnxthxqv8fvdcn6f4xhxwa7ms729ak3gsl4qrq2mm
    value: 1000000
```

### Starting the node

Now, for starting the node and be able to generate new blocks, you have to put your pool's private keys and id in a file and start the node with the `--secret filename` parameter.

---

For example, if you follow the examples of the [registering stake pool guide](../stake_pool/registering_stake_pool.md) 

You could create a file called poolsecret.yaml with the following content.

```yaml
genesis:
  sig_key: Content of stake_pool_kes.prv file
  vrf_key: Content of stake_pool_vrf.prv file
  node_id: Content of stake_pool.id file
```

And you could start the node with this command 

```sh
jormungandr --genesis-block block-0.bin --config config.yaml --secret poolsecret.yaml
```

# Test script

There is a script [here](https://github.com/input-output-hk/jormungandr/blob/master/scripts/bootstrap) that can be used to bootstrap a test node with a pre-set faucet and stake pool and can be used as an example.


