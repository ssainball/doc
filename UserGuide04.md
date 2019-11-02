# jcli

This is the node command line helper. It is mostly meant for developers and
stake pool operators. It allows offline operations:

* generating cryptographic materials for the wallets and stake pools;
* creating addresses, transactions and certificates;
* prepare a new blockchain

and it allows simple interactions with the node:

* query stats;
* send transactions and certificates;
* get raw blocks and UTxOs.


# cryptographic keys

There are multiple type of key for multiple use cases.

| type | usage |
|:----:|:------|
|`Ed25519` | Signing algorithm for Ed25519 algorithm |
|`Ed25519Bip32`| Related to the HDWallet, Ed25519 Extended with chain code for derivation derivation |
|`Ed25519Extended`| Related to `Ed25519Bip32` without the chain code |
|`SumEd25519_12`| For stake pool, necessary for the KES |
|`Curve25519_2HashDH`| For stake pool, necessary for the VRF |


There is a command line parameter to generate this keys:

```
$ jcli key generate --type=Ed25519
ed25519_sk1cvac48ddf2rpk9na94nv2zqhj74j0j8a99q33gsqdvalkrz6ar9srnhvmt
```

and to extract the associated public key:

```
$ echo ed25519_sk1cvac48ddf2rpk9na94nv2zqhj74j0j8a99q33gsqdvalkrz6ar9srnhvmt | jcli key to-public
ed25519_pk1z2ffur59cq7t806nc9y2g64wa60pg5m6e9cmrhxz9phppaxk5d4sn8nsqg
```

## Signing data

Sign data with private key. Supported key formats are: ed25519, ed25519bip32, ed25519extended and
sumed25519_12.

```
jcli key sign <options> <data>
```

The options are
- --secret-key <secret_key> - path to file with bech32-encoded secret key
- -o, --output <output> - path to file to write signature into, if no value is passed,
standard output will be used

<data> - path to file with data to sign, if no value is passed, standard input will be used


## Verifying signed data

Verify signed data with public key. Supported key formats are: ed25519, ed25519bip32 and
sumed25519_12.

```
jcli key verify <options> <data>
```

The options are
- --public-key <public_key> - path to file with bech32-encoded public key
- --signature <signature> - path to file with signature

<data> - path to file with data to sign, if no value is passed, standard input will be used



# Address

Jormungandr comes with a separate CLI to create and manipulate addresses.

This is useful for creating addresses from their components in the CLI,
for debugging addresses and for testing.

## Display address info

To display an address and verify it is in a valid format you can utilise:

```
$ jcli address info ta1svy0mwwm7mdwcuj308aapjw6ra4c3e6cygd0f333nvtjzxg8ahdvxlswdf0
discrimination: testing
public key: ed25519e_pk1pr7mnklkmtk8y5tel0gvnksldwywwkpzrt6vvvvmzus3jpldmtpsx9rnmx
```

or for example:

```
$ jcli address \
    info \
    ca1qsy0mwwm7mdwcuj308aapjw6ra4c3e6cygd0f333nvtjzxg8ahdvxz8ah8dldkhvwfghn77se8dp76uguavzyxh5cccek9epryr7mkkr8n7kgx
discrimination: production
public key: ed25519_pk1pr7mnklkmtk8y5tel0gvnksldwywwkpzrt6vvvvmzus3jpldmtpsx9rnmx
group key:  ed25519_pk1pr7mnklkmtk8y5tel0gvnksldwywwkpzrt6vvvvmzus3jpldmtpsx9rnmx
```

## Creating an address

Each command following allows to create addresses for production and testing
chains. For chains, where the discrimination is `testing`, you need to
use the `--testing` flag.

There's 3 types of addresses:

* Single address : A simple spending key. This doesn't have any stake in the system
* Grouped address : A spending key attached to an account key. The stake is automatically
* Account address : An account key. The account is its own stake

### Address for UTxO

You can create a single address (non-staked) using the spending public key for
this address utilising the following command:

```
$ jcli address \
    single ed25519e_pk1jnlhwdgzv3c9frknyv7twsv82su26qm30yfpdmvkzyjsdgw80mfqduaean
ca1qw207ae4qfj8q4yw6v3ned6psa2r3tgrw9u3y9hdjcgj2p4pcaldyukyka8
```

To add the staking information and make a group address, simply add the account
public key as a second parameter of the command:

```
$ jcli address \
    single \
    ed25519_pk1fxvudq6j7mfxvgk986t5f3f258sdtw89v4n3kr0fm6mpe4apxl4q0vhp3k \
    ed25519_pk1as03wxmy2426ceh8nurplvjmauwpwlcz7ycwj7xtl9gmx9u5gkqscc5ylx
ca1q3yen35r2tmdye3zc5lfw3x992s7p4dcu4jkwxcda80tv8xh5ym74mqlzudkg42443nw08cxr7e9hmcuzals9ufsa9uvh723kvteg3vpvrcxcq
```

### Address for Account

To create an account address you need the account public key and run:

```
$ jcli address \
    account ed25519_pk1c4yq3hflulynn8fef0hdq92579n3c49qxljasrl9dnuvcksk84gs9sqvc2
ca1qhz5szxa8lnujwva8997a5q42nckw8z55qm7tkq0u4k03nz6zc74ze780qe
```

### changing the address prefix

You can decide to change the address prefix, allowing you to provide more
enriched data to the user. However, this prefix is not forwarded to the node,
it is only for UI/UX.

```
$ jcli address \
    account \
    --prefix=address_ \
    ed25519_pk1yx6q8rsndawfx8hjzwntfs2h2c37v5g6edv67hmcxvrmxfjdz9wqeejchg
address_1q5smgquwzdh4eyc77gf6ddxp2atz8ej3rt94nt6l0qes0vexf5g4cw68kdx
```


# transaction

Tooling for offline transaction creation and signing.

```
jcli transaction
```

Those familiar with [`cardano-cli`](http://github.com/input-output-hk/cardano-cli)
transaction builder will see resemblance in `jcli transaction`.

There is a couple of commands that can be used to:

1. prepare a transaction:
    - `new` create a new empty transaction;
    - `add-input`
    - `add-account`
    - `add-output`
2. `finalize` the transaction for signing:
3. create witnesses and add the witnesses:
    - `make-witness`
    - `add-witness`
4. `seal` the transaction, ready to send to the blockchain

There are also functions to help decode and display the
content information of a transaction:

* `info`
* `id` to get the **Transaction ID** of the transaction
* `to-message` to get the hexadecimal encoded message, ready to send with `cli rest message`


# Examples

The following example focuses on using an utxo as input, the few differences when transfering from an account will be pointed out when necessary.
There is also a script [here](https://github.com/input-output-hk/jormungandr/blob/master/scripts/send-transaction) to send a transaction from a faucet account to a specific address which could be used as a reference.

Let's use the following utxo as input and transfer 50 lovelaces to the destination address

## Input utxo

| Field                     | Value        |
| ------------------------- |:------------:|
| UTXO's transaction ID     | 55762218e5737603e6d27d36c8aacf8fcd16406e820361a8ac65c7dc663f6d1c| 
| UTXO's output index       | 0     | 
| associated address        |  ca1q09u0nxmnfg7af8ycuygx57p5xgzmnmgtaeer9xun7hly6mlgt3pjyknplu    | 
| associated value          | 100             |

## Destination address

**address**: ca1qvnr5pvt9e5p009strshxndrsx5etcentslp2rwj6csm8sfk24a2wlqtdj6

## Create a staging area

```sh
jcli transaction new > tx
```

## Add input

For the input, we need to reference the uxto with the **UTXO's transaction ID** and **UTXO'S output index** fields and we need to specify how much coins are there with the **associated value** field.

### Example - UTXO address as Input

```sh
jcli transaction add-input  55762218e5737603e6d27d36c8aacf8fcd16406e820361a8ac65c7dc663f6d1c 0 100 --staging tx
```

### Example - Account address as Input

If the input is an account, the command is slightly different

```sh
jcli transaction add-account account_address account_funds --staging tx
```

## Add output

For the output, we need the address we want to transfer to, and the amount.

```sh
jcli transaction add-output  ca1qvnr5pvt9e5p009strshxndrsx5etcentslp2rwj6csm8sfk24a2wlqtdj6 50 --staging tx
```

## Add fee and change address

We want to get the change in the same address that we are sending from (the *associated address* of the utxo). We also specify how to compute the fees.
You can leave out the `--fee-constant 5 --fee-coefficient 2` part if those are both 0.

```sh
jcli transaction finalize  ca1q09u0nxmnfg7af8ycuygx57p5xgzmnmgtaeer9xun7hly6mlgt3pjyknplu --fee-constant 5 --fee-coefficient 2 --staging tx
```

Now, if you run

```sh
jcli transaction info --fee-constant 5 --fee-coefficient 2 --staging tx
```

You should see something like this

```plaintext
Transaction `0df39a87d3f18a188b40ba8c203f85f37af665df229fb4821e477f6998864273' (finalizing)
  Input:   100
  Output:  89
  Fees:    11
  Balance: 0
 - 55762218e5737603e6d27d36c8aacf8fcd16406e820361a8ac65c7dc663f6d1c:0 100
 + ca1qvnr5pvt9e5p009strshxndrsx5etcentslp2rwj6csm8sfk24a2wlqtdj6 50
 + ca1q09u0nxmnfg7af8ycuygx57p5xgzmnmgtaeer9xun7hly6mlgt3pjyknplu 39
```

## Sign the transaction

### Make witness

For signing the transaction, you need the private key associated with the input address (the one that's in the utxos) and the hash of the genesis block of the network you are connected to.

The genesis' hash is needed for ensuring that the transaction cannot be re-used in another blockchain and for security concerns on offline transaction signing, as we are signing the transaction for the specific blockchain started by this block0 hash.

The following command takes the private key in the *key.prv* file and creates a witness in a file named *witness* in the current directory. 

```sh
jcli transaction make-witness --genesis-block-hash abcdef987654321... --type utxo txid witness key.prv
```

---
#### Account input

When using an account as input, the command takes `account` as the type and an additional parameter: `--account-spending-counter`, that should be increased every time the account is used as input.

e.g.

```sh
jcli transaction make-witness --genesis-block-hash abcdef987654321... --type account --account-spending-counter 0 witness key.prv
```

### Add witness

```sh
jcli transaction add-witness witness --staging tx
```

## Send the transaction

```sh
jcli transaction seal --staging tx
```

```sh
jcli transaction to-message --staging tx > txmsg
```

Send it using the rest api

```sh
jcli rest v0 message post -f txmsg --host http://127.0.0.1:8443/api
```

## Checking if the transaction was accepted

You can check if the transaction was accepted by checking the node logs, for example, if the transaction is accepted 

`jcli rest v0 message logs -h http://127.0.0.1:8443/api`

```plaintext
---
- fragment_id: d6ef0b2148a51ed64531efc17978a527fd2d2584da1e344a35ad12bf5460a7e2
  last_updated_at: "2019-06-11T15:38:17.070162114Z"
  received_at: "2019-06-11T15:37:09.469101162Z"
  received_from: Rest
  status:
    InABlock:
      date: "4.707"
      block: "d9040ca57e513a36ecd3bb54207dfcd10682200929cad6ada46b521417964174"
```

Where the InABlock status means that the transaction was accepted in the block with date "4.707"
and for block `d9040ca57e513a36ecd3bb54207dfcd10682200929cad6ada46b521417964174`.

The status here could also be: 

Pending: if the transaction is received and is pending being added in the blockchain (or rejected).

or

Rejected: with an attached message of the reason the transaction was rejected. 



# certificate

Tooling for offline transaction creation

## Builder

Builds a signed certificate.

The process can be split into steps on the first step certificate
is created.
```
jcli certificate new stake-pool-registration \
  --vrf-key <vrf-public-key> --kes-key <kes-public-key> \
  [--owner <owner-public-key>] \
  --serial <node-serial> \
  <output-file>
```

if output-file is omited result will be written to stdout. Once
certificate is ready you must sign it with the private keys of
all the owners:


```
jcli certificate sign <key> <input-file> <output-file>
```



# Genesis

Tooling for working with a genesis file

# Usage
```sh
jcli genesis [subcommand]
```

## Subcommands

- decode: Print the YAML file corresponding to an encoded genesis block.
- encode: Create the genesis block of the blockchain from a given yaml file.
- hash: Print the block hash of the genesis 
- init: Create a default Genesis file with appropriate documentation to help creating the YAML file
- help

## Examples

### Encode a genesis file

```sh
jcli genesis encode --input genesis.yaml --output block-0.bin
```

or equivantely

```sh
cat genesis.yaml | jcli genesis encode > block-0.bin
```

### Get the hash of an encoded genesis file

```sh
jcli genesis hash --input block-0.bin
```


# Genesis

Tooling for working with a genesis file

# Usage
```sh
jcli genesis [subcommand]
```

## Subcommands

- decode: Print the YAML file corresponding to an encoded genesis block.
- encode: Create the genesis block of the blockchain from a given yaml file.
- hash: Print the block hash of the genesis 
- init: Create a default Genesis file with appropriate documentation to help creating the YAML file
- help

## Examples

### Encode a genesis file

```sh
jcli genesis encode --input genesis.yaml --output block-0.bin
```

or equivantely

```sh
cat genesis.yaml | jcli genesis encode > block-0.bin
```

### Get the hash of an encoded genesis file

```sh
jcli genesis hash --input block-0.bin
```
