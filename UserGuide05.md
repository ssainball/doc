# Staking with jormungandr

Here we will describe how to:

* _delegate your stake to a stake pool_ so that you can participate to the consensus
  and maybe collect rewards for that;

If you are a user of the blockchain then you will certainly not need to look
at the last section about _how to register a stake pool_.


``다음은 방법을 설명합니다.``

*  ``컨센서스에 참여하고 이에 대한 보상을 받을 수 있도록 스테이크 풀에 스테이크를 위임하십시오.``

``블록체인 사용자 인 경우 스테이크 풀을 등록하는 방법에 대한 마지막 섹션을 반드시 볼 필요는 없습니다.``


# Delegating your stake


## how to create the delegation certificate

Stake is concentrated in accounts, and you will need your account key to
delegate its associated stake.

``스테이크는 계정에 집중되어 있으며 관련 스테이크를 위임하려면 계정 키가 필요합니다.``

You will need your:

* account public key: a bech32 string of a public key
* the Stake Pool ID: an hexadecimal string identifying the stake pool you want
  to delegate your stake to.

"당신만의 암호화 자료가 필요합니다" :

* ``계정 공개 키 : 공개 키의 bech32 문자열``
* ``스테이크 풀 ID : 스테이크를 위임 할 스테이크 풀을 식별하는 16 진수 문자열.``

```
$ jcli certificate new stake-delegation STAKE_POOL_ID ACCOUNT_PUBLIC_KEY > stake_delegation.cert
```

## how to sign your delegation certificate

We need to make sure that the owner of the account is authorizing this
delegation to happens, and for that we need a cryptographic signature.

``계정 소유자가 이 위임을 승인하고 암호화 서명이 필요한지 확인해야합니다.``

We will need the account secret key to create a signature

``서명을 만들려면 계정 비밀 키가 필요합니다``

```
$ cat stake_delegation.cert | jcli certificate sign account_key.prv | tee stake_delegation.cert
cert1q8rv4ccl54k99rtnm39...zr0
```

The output can now be added in the `transaction` and submitted to a node.

``이제 출력을 트랜잭션에 추가하고 노드에 제출할 수 있습니다.``

## submitting to a node

The `jcli transaction add-certificate` command should be used to add a certificate **before finalizing** the transaction.

``jcli transaction add-certificate 명령을 사용하여 트랜잭션을 완료하기 전에 인증서를 추가해야합니다.``

For example:

```sh

...

jcli transaction add-certificate $(cat stake_delegation.cert) --staging tx

jcli transaction finalize CHANGE_ADDRESS --fee-constant 5 --fee-coefficient 2 --fee-certificate 2 --staging tx

...

```

The `--fee-certificate` flag indicates the cost of adding a certificate, used for computing the fees, it can be omitted if it is zero.

``--fee-certificate 플래그는 요금을 계산하는 데 사용되는 인증서 추가 비용을 나타내며 0이면 생략 할 수 있습니다.``

See [here](../jcli/transaction.md) for more documentation on transaction creation.

``트랜잭션 생성에 대한 자세한 내용은 여기를 참조하십시오.``


# registering stake pool

There are multiple components to be aware of when running a stake pool:

`` 스테이크 풀을 실행할 때 알아야 할 여러 구성 요소가 있습니다. ``

* your `NodeId`: 블록체인 프로토콜 내의 식별자입니다 (지갑은 이 NodeId를 통해 스테이크 풀에 위임합니다);
* your [**VRF**] key pairs: 리더 선거에 참여하기 위해 사용할 암호화 자료입니다;
* your [**KES**] key pairs: 블록에 서명하는 데 사용할 암호화 자료입니다.

So in order to start your stake pool you will need to generate these objects.

``따라서 스테이크 풀을 시작하려면 이러한 객체를 생성해야합니다.``

# The primitives

## VRF key pair

To generate your [**VRF**] Key pairs, we will utilise [`jcli`] as described
[here](../jcli/key.md):

``VRF 키 페어를 생성하기 위해 다음에 설명 된대로 [jcli]를 사용합니다.``

```
$ jcli key generate --type=Curve25519_2HashDH > stake_pool_vrf.prv
```

`stake_pool_vrf.prv` file now contains the VRF private key.

``stake_pool_vrf.prv 파일에 VRF 개인 키가 포함되었습니다.``

```
$ cat stake_pool_vrf.prv | jcli key to-public > stake_pool_vrf.pub
```

## KES key pair

Similar to above:

```
$ jcli key generate --type=SumEd25519_12 > stake_pool_kes.prv
```

`stake_pool_kes.prv` now contains your KES private key

``stake_pool_kes.prv는 이제 KES 개인 키를 포함합니다``

```
$ cat stake_pool_kes.prv | jcli key to-public > stake_pool_kes.pub
```

# creating your stake pool certificate

The certificate is what will be sent to the blockchain in order to register
yourself to the other participants of the blockchain that you are a stake
pool too.


``인증서는 귀하가 스테이크 풀이라는 다른 블록 체인 참가자에게 자신을 등록하기 위해 블록 체인으로 전송되는 것입니다.``

```
$ jcli certificate new stake-pool-registration \
    --kes-key $(cat stake_pool_kes.pub) \
    --vrf-key $(cat stake_pool_vrf.pub) \
    --serial 1010101010 > stake_pool.cert
```

Now you need to sign this certificate with the owner key:

``이제 소유자 키로 이 인증서에 서명해야합니다.``

```
$ cat stake_pool.cert | jcli certificate sign stake_key.prv | tee stake_pool.cert
cert1qsqqqqqqqqqqqqqqqqqqq0p5avfqp9tzusr26...cegxaz
```

And now you can retrieve your stake pool id (`NodeId`):

``이제 스테이크 풀 ID (NodeId)를 검색 할 수 있습니다.``

```
$ cat stake_pool.cert | jcli certificate get-stake-pool-id | tee stake_pool.id
ea830e5d9647af89a5e9a4d4089e6e855891a533316adf4a42b7bf1372389b74
```

[**VRF**]: https://en.wikipedia.org/wiki/Verifiable_random_function



# Advanced

This section is meant for advanced users and developers of the node, or if
you to learn more about the node.

``이 섹션은 노드의 고급 사용자 및 개발자를 위해 또는 노드에 대해 더 많이 배우려는 경우에 사용됩니다.``

At the moment, it only covers details on how to create your own blockchain genesis
configuration, but in normal case, the blockchain configuration should be available
with the specific blockchain system.

``현재로서는 자체 블록체인 생성 구성을 만드는 방법에 대해서만 자세히 설명하지만 일반적인 경우 특정 블록체인 시스템에서 블록 체인 구성을 사용할 수 있어야합니다.``


# genesis file

The genesis file is the file that allows you to create a new blockchain
from block 0. It lays out the different parameter of your blockchain:
the initial utxo, the start time, the slot duration time, etc...

``생성 파일은 블록 0에서 새 블록 체인을 생성 할 수있는 파일입니다. 블록 체인의 다른 매개 변수를 배치합니다 : 초기 utxo, 시작 시간, 슬롯 지속 시간 등.``

Example of a BFT genesis file with an initial address UTxO and an account UTxO.
More info regarding [starting a BFT blockchain here](./02_starting_bft_blockchain.md)
and [regarding addresses there](../jcli/address.md).
You could also find information regarding the [jcli genesis tooling](../jcli/genesis.md).

``초기 주소 UTxO 및 계정 UTxO를 가진 BFT 생성 파일의 예 여기에서 BFT 블록 체인 시작 및 주소에 관한 추가 정보. jcli 생성 툴링에 관한 정보도 찾을 수 있습니다.``

You can generate a documented pre-generated genesis file:

``문서화 된 사전 생성 된 생성 파일을 생성 할 수 있습니다.``

```
jcli genesis init
```

For example your genesis file may look like:

``예를 들어, 생성 파일은 다음과 같습니다.``

```yaml
{{#include ../../jormungandr-lib/src/interfaces/block0_configuration/DOCUMENTED_EXAMPLE.yaml}}
```

There are multiple _parts_ in the genesis file:

``생성 파일에는 여러 부분이 있습니다.``

* `blockchain_configuration`: 이것은 블록 체인의 구성 파라미터 목록이며, 일부는 업데이트 프로토콜을 통해 나중에 변경 될 수 있습니다.;
* `initial`: list of steps to create initial state of ledger. ``(원장 초기 상태 생성 단계의 리스트)``

## `blockchain_configuration` options

| option | format | description |
|:-------|:-------|:------------|
| `block0_date` | number | EPOCH 시작 이후의 초 단위, 블록체인 공식 시작 시간 |
| `discrimination` | string | `production` or `test` |
| `block0_consensus` | string | `bft` |
| `slot_duration` | number | 2개의 블록 생성 사이의 시간(초) |
| `epoch_stability_depth` | number | allowed size of a fork (in number of block) |
| `consensus_leader_ids` | array | BFT 리더 목록 (블록체인 처음 시작 할 때) |
| `max_number_of_transactions_per_block` | number | 블록에서 허용되는 최대 트랜잭션 수 |
| `bft_slots_ratio` | number | placeholder(자리표시자), do not use(사용금지) |
| `linear_fees` | object | linear fee settings, 거래 및 인증서 게시 수수료 설정 |
| `consensus_genesis_praos_active_slot_coeff` | number | 기원 praos 활성 슬롯 계수. 슬롯 리더가 되기 위해 필요한 최소 스테이크 결정, 범위 (0,1)에 있어야 함 |
| `kes_update_speed` | number | KES 키를 업데이트 하는 속도, 초단위 |
| `slots_per_epoch` | number | 각 에포크의 슬롯 수 |

_for more information about the BFT leaders in the genesis file, see
[Starting a BFT Blockchain](./02_starting_bft_blockchain.md)_

## `initial` options

Each entry can be one of 3 variants:

| variant | format | description |
|:-------|:-------|:------------|
| `fund` | sequence | 블록 체인의 초기 예치금 (항목 당 최대 255 개의 출력) |
| `cert` | string | initial certificate (초기증명서 |
| `legacy_fund` | sequence | 펀드와 동일하지만 기존 Cardano 주소 형식 |

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
| `value` | number | assigned value (할당된 가치) |

`legacy_fund` differs only in address format, which is legacy Cardano

``legacy_fund 는 legacy Cardano 주소 형식만 다릅니다.


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

``이제 블록 체인이 초기화되었으므로 노드를 시작해야합니다.``

Write you private key in a file on your HD:

``HD의 파일에 개인 키를 작성하십시오.``

``

```
$ cat node_secret.yaml
bft:
  signing_key: ed25519_sk1hpvne...
```

Configure your Node (config.yml) and run the following command:

``노드 (config.yml)를 구성하고 다음 명령을 실행하십시오.``

```
$ jormungandr --genesis-block block-0.bin \
    --config example.config \
    --secret node_secret.yaml
```

It's possible to use the flag `--secret` multiple times to run a node
with multiple leaders.

``여러 리더와 함께 노드를 실행하기 위해 --secret 플래그를 여러 번 사용할 수 있습니다.``

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

``또한 여기에는 bft 합의 프로토콜로 테스트 노드를 부트 스트랩하는 데 사용할 수있는 스크립트가 있습니다.``

# starting a genesis blockchain

When starting a genesis praos blockchain there is an element to take
into consideration while constructing the block 0: _the stake distribution_.

``창세기 praos 블록 체인을 시작할 때 블록 0을 구성하는 동안 고려해야 할 요소 인 스테이크 분포가 있습니다.``

In the context of Genesis/Praos the network is fully decentralized and it is
necessary to think ahead about initial stake pools and to make sure there
is stake delegated to these stake pools.

``Genesis / Praos의 맥락에서 네트워크는 완전히 분산되어 있으며 초기 스테이크 풀에 대해 미리 생각하고 스테이크 풀에 위임 된 스테이크가 있는지 확인해야합니다.``

In your genesis yaml file, make sure to set the following values to the appropriate
values/desired values:

``생성 yaml 파일에서 다음 값을 적절한 값 / 원하는 값으로 설정하십시오.``

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

``block0_consensus를 genesis_praos로 설정하면 기원 praos를 합의 레이어로 사용하여 블록 체인을 시작하려고합니다.``

`bft_slots_ratio` needs to be set to `0` (we don't support composite modes between
BFT mode and Genesis mode -- yet).

``bft_slots_ratio는 0으로 설정해야합니다 (BFT 모드와 Genesis 모드 사이의 복합 모드는 지원하지 않습니다).``

`consensus_genesis_praos_active_slot_coeff` determines minimum stake required to
try becoming slot leader, must be in range 0 exclusive and 1 inclusive.

``consensus_genesis_praos_active_slot_coeff는 슬롯 리더가되기 위해 필요한 최소 스테이크를 결정합니다. 범위는 0과 1 사이 여야합니다.``

## The initial certificates

In the `initial_certs` field you will set the initial certificate. This is important
to declare the stake pool and delegate stake to them. Otherwise no block will be ever
created.

``initial_certs 필드에서 초기 인증서를 설정합니다. 스테이크 풀을 선언하고 스테이크를 위임하는 것이 중요합니다. 그렇지 않으면 블록이 생성되지 않습니다.``

Remember that in this array the **order** matters:

``이 배열에서 순서는 중요합니다.``

In order to delegate your stake, you need a stake pool to already exist, so the stake pool registration certificate should go first.

``스테이크를 위임하려면 스테이크 풀이 이미 존재해야하므로 스테이크 풀 등록 인증서가 먼저 있어야합니다.``

### Stake pool registration

Now you can register a stake pool.
Follow the instruction in [registering stake pool guide](../stake_pool/registering_stake_pool.md).

``이제 스테이크 풀을 등록 할 수 있습니다. 스테이크 풀 가이드 등록 지침을 따르십시오.``

The _owner key_ (the key you sign the stake pool registration certificate) is the secret
key associated to a previously registered stake key.

``소유자 키 (스테이크 풀 등록 인증서에 서명 한 키)는 이전에 등록 된 스테이크 키와 관련된 비밀 키입니다.``

### Delegating stake

Now that there is both your stake key and there are stake pools available
in the block0 you need to delegate to one of the stake pool. Follow the instruction
in [delegating stake](../stake_pool/delegating_stake.md).

``이제 스테이크 키가 있고 블록 0에 사용 가능한 스테이크 풀이 있으므로 스테이크 풀 중 하나에 위임해야합니다. 스테이크 위임의 지시를 따르십시오.``

And in the initial funds start adding the addresses. To create an address with delegation
follow the instruction in [JCLI's address guide](../jcli/address.md). Utilise the stake key
registered previously as group address:

``그리고 초기 자금에서 주소 추가를 시작하십시오. 위임 된 주소를 작성하려면 JCLI의 주소 안내서에있는 지시 사항을 따르십시오. 이전에 그룹 주소로 등록 된 스테이크 키를 사용하십시오.``

```
jcli address single $(cat wallet_key.pub) $(cat stake_key.pub)
ta1sjx4j3jwel94g0cgwzq9au7h6m8f5q3qnyh0gfnryl3xan6qnmjse3k2uv062mzj34eacjnxthxqv8fvdcn6f4xhxwa7ms729ak3gsl4qrq2mm
```

You will notice that addresses with delegation are longer (about twice longer) than
address without delegation.

``위임이 있는 주소가 위임이 없는 주소보다 깁니다(약 2배).``

For example, the most minimal setting you may have is:

``예를 들어, 가장 작은 설정은 다음과 같습니다.``

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

``이제 노드를 시작하고 새 블록을 생성하려면 풀의 개인 키와 ID를 파일에 넣고 --secret filename 매개 변수로 노드를 시작해야합니다.``

---

For example, if you follow the examples of the [registering stake pool guide](../stake_pool/registering_stake_pool.md) 

``예를 들어 스테이크 풀 등록 가이드의 예를 따르면``

You could create a file called poolsecret.yaml with the following content.

``다음 내용으로 poolsecret.yaml이라는 파일을 만들 수 있습니다.``

```yaml
genesis:
  sig_key: Content of stake_pool_kes.prv file
  vrf_key: Content of stake_pool_vrf.prv file
  node_id: Content of stake_pool.id file
```

And you could start the node with this command 

``이 명령으로 노드를 시작할 수 있습니다``

```sh
jormungandr --genesis-block block-0.bin --config config.yaml --secret poolsecret.yaml
```

# Test script

There is a script [here](https://github.com/input-output-hk/jormungandr/blob/master/scripts/bootstrap) that can be used to bootstrap a test node with a pre-set faucet and stake pool and can be used as an example.

``여기에는 사전 설정된 facuet 및 스테이크 풀이 있는 테스트 노드를 부트 스트랩하는 데 사용할 수 있는 스크립트가 있으며 예제로 사용할 수 있습니다.``
