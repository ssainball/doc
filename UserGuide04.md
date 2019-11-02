# jcli

This is the node command line helper. It is mostly meant for developers and
stake pool operators. It allows offline operations:

``jcli는 노드 명령 행 헬퍼입니다. 주로 개발자와 스테이크 풀 운영자를위한 것입니다. 오프라인 작업을 허용합니다.``

* generating cryptographic materials for the wallets and stake pools;
* creating addresses, transactions and certificates;
* prepare a new blockchain

``* 지갑 및 스테이크 풀을 위한 암호화 자료를 생성하는 단계``

``* 주소, 거래, 인증서 작성``

``* 새로운 블록 체인 준비``

and it allows simple interactions with the node:

``노드와의 간단한 상호 작용을 허용합니다.``

* query stats;
* send transactions and certificates;
* get raw blocks and UTxOs.

``* 쿼리 통계``

``* 거래 및 인증서 보내기``

``* raw blocks 과 UTxO 얻기``

# cryptographic keys

There are multiple type of key for multiple use cases.

``여러 사용 사례에 대한 다양한 유형의 키가 있습니다.``

| 유형 | 사용법 |
|:----:|:------|
|`Ed25519` | 알고리즘의 서명 알고리즘 |
|`Ed25519Bip32`| HDWallet 관련, 체인코드로 확장된 HDWallet |
|`Ed25519Extended`| 체인 코드없이 Ed25519Bip32와 관련됨 |
|`SumEd25519_12`| 스테이크 풀의 경우 KES 키 생성에 사용됨 |
|`Curve25519_2HashDH`| 스테이크 풀의 경우 VRF 키 생성에 사용됨 |


There is a command line parameter to generate this keys:

``이 키를 생성하는 명령 행 매개 변수가 있습니다.``

```
$ jcli key generate --type=Ed25519
ed25519_sk1cvac48ddf2rpk9na94nv2zqhj74j0j8a99q33gsqdvalkrz6ar9srnhvmt
```

and to extract the associated public key:

``관련 공개 키를 추출합니다.``

```
$ echo ed25519_sk1cvac48ddf2rpk9na94nv2zqhj74j0j8a99q33gsqdvalkrz6ar9srnhvmt | jcli key to-public
ed25519_pk1z2ffur59cq7t806nc9y2g64wa60pg5m6e9cmrhxz9phppaxk5d4sn8nsqg
```

## Signing data

Sign data with private key. Supported key formats are: ed25519, ed25519bip32, ed25519extended and
sumed25519_12.

``개인 키로 데이터에 서명하십시오. 지원되는 키 형식은 ed25519, ed25519bip32, ed25519extended 및 sumed25519_12입니다.``

```
jcli key sign <options> <data>
```

The options are
- --secret-key <secret_key> - path to file with bech32-encoded secret key
- -o, --output <output> - path to file to write signature into, if no value is passed,
standard output will be used

``--secret-key <secret_key> : bech32로 인코딩 된 비밀 키가 있는 파일 경로``

``-o, --output <output> : 서명을 쓸 파일의 경로입니다. 값이 전달되지 않으면 표준 출력이 사용됩니다.``

<data> - path to file with data to sign, if no value is passed, standard input will be used

``-서명 할 데이터가 있는 파일 경로, 값이 전달되지 않으면 표준 입력이 사용됩니다.``

## Verifying signed data

Verify signed data with public key. Supported key formats are: ed25519, ed25519bip32 and
sumed25519_12.

``공개 키로 서명 된 데이터를 확인하십시오. 지원되는 키 형식은 ed25519, ed25519bip32 및 sumed25519_12입니다.``

```
jcli key verify <options> <data>
```

The options are
- --public-key <public_key> - path to file with bech32-encoded public key
- --signature <signature> - path to file with signature

''--public-key <public_key> : bech32로 인코딩 된 공개 키가있는 파일 경로''

``--signature <signature> : 서명이있는 파일의 경로``

<data> - path to file with data to sign, if no value is passed, standard input will be used

``-서명 할 데이터가 있는 파일 경로, 값이 전달되지 않으면 표준 입력이 사용됩니다.``


# Address

Jormungandr comes with a separate CLI to create and manipulate addresses.

``Jormungandr에는 주소를 생성하고 조작하기 위한 별도의 CLI가 제공됩니다.``

This is useful for creating addresses from their components in the CLI,
for debugging addresses and for testing.

``이는 CLI의 구성 요소에서 주소를 작성하고 주소를 디버깅하고 테스트하는 데 유용합니다.``


## Display address info

To display an address and verify it is in a valid format you can utilise:

``주소를 표시하고 유효한 형식인지 확인하려면 다음을 사용할 수 있습니다.``

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

``다음의 각 명령을 사용하면 운영중인 블록체인 또는 테스트용 블록체인 에 대한 주소를 만들 수 있습니다. 테스트를 하는 경우 --testing 플래그를 사용해야합니다.``

There's 3 types of addresses:

* Single address : A simple spending key. This doesn't have any stake in the system
* Grouped address : A spending key attached to an account key. The stake is automatically
* Account address : An account key. The account is its own stake

``3 가지 유형의 주소가 있습니다.``

``* 단일 주소 : 간단한 지출 키. 이 시스템에 아무런 지분이 없습니다``

``* 그룹화 된 주소 : 계정 키에 연결된 지출 키. 스테이크는 자동으로``

``* 계정 주소 : 계정 키. 계정 자체가 지분입니다``


### Address for UTxO

You can create a single address (non-staked) using the spending public key for
this address utilising the following command:

``다음 명령을 사용하여 지출 공개 키를 사용하여 단일 주소(스테이크되지 않은)를 만들 수 있습니다.``

```
$ jcli address \
    single ed25519e_pk1jnlhwdgzv3c9frknyv7twsv82su26qm30yfpdmvkzyjsdgw80mfqduaean
ca1qw207ae4qfj8q4yw6v3ned6psa2r3tgrw9u3y9hdjcgj2p4pcaldyukyka8
```

To add the staking information and make a group address, simply add the account
public key as a second parameter of the command:

``스테이 킹 정보를 추가하고 그룹 주소를 만들려면 계정 공개 키를 명령의 두 번째 매개 변수로 추가하십시오.``

```
$ jcli address \
    single \
    ed25519_pk1fxvudq6j7mfxvgk986t5f3f258sdtw89v4n3kr0fm6mpe4apxl4q0vhp3k \
    ed25519_pk1as03wxmy2426ceh8nurplvjmauwpwlcz7ycwj7xtl9gmx9u5gkqscc5ylx
ca1q3yen35r2tmdye3zc5lfw3x992s7p4dcu4jkwxcda80tv8xh5ym74mqlzudkg42443nw08cxr7e9hmcuzals9ufsa9uvh723kvteg3vpvrcxcq
```

### Address for Account

To create an account address you need the account public key and run:

``계정 주소를 만들려면 계정 공개 키가 필요합니다.``

```
$ jcli address \
    account ed25519_pk1c4yq3hflulynn8fef0hdq92579n3c49qxljasrl9dnuvcksk84gs9sqvc2
ca1qhz5szxa8lnujwva8997a5q42nckw8z55qm7tkq0u4k03nz6zc74ze780qe
```

### changing the address prefix

You can decide to change the address prefix, allowing you to provide more
enriched data to the user. However, this prefix is not forwarded to the node,
it is only for UI/UX.

``주소 접두사를 변경하여 보다 풍부한 데이터를 사용자에게 제공 할 수 있습니다. 그러나이 접두부는 노드로 전달되지 않으며 UI/UX 전용입니다.``

```
$ jcli address \
    account \
    --prefix=address_ \
    ed25519_pk1yx6q8rsndawfx8hjzwntfs2h2c37v5g6edv67hmcxvrmxfjdz9wqeejchg
address_1q5smgquwzdh4eyc77gf6ddxp2atz8ej3rt94nt6l0qes0vexf5g4cw68kdx
```


# transaction

Tooling for offline transaction creation and signing.

``오프라인 트랜잭션 생성 및 서명을 위한 툴링.``

```
jcli transaction
```

Those familiar with [`cardano-cli`](http://github.com/input-output-hk/cardano-cli)
transaction builder will see resemblance in `jcli transaction`.

``cardano-cli transaction builder에 익숙한 사람들은 jcli transaction과 유사합니다.``

There is a couple of commands that can be used to:

``다음과 같은 명령을 사용할 수 있습니다.``

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


``1. 거래 준비: ``
``  - 비어있는 새 트랜잭션을 새로 작성하십시오.``
``  - add-input``
``  - add-account``
``  - add-output``

``2. 서명을 위한 트랜잭션 마무리합니다 :``

``3. 증인을 만들고 증인을 추가하십시오.``
``  - make-witness``
``  - add-witness``

``4. 거래를 봉인하고 블록 체인에 보낼 준비가 되었습니다.``


There are also functions to help decode and display the
content information of a transaction:

``트랜잭션의 컨텐츠 정보를 디코딩하고 표시하는 데 도움이되는 기능도 있습니다.``

* `info`
* `id` to get the **Transaction ID** of the transaction
* `to-message` to get the hexadecimal encoded message, ready to send with `cli rest message`

``정보``

``거래의 거래 ID를 얻기위한 id``

``16 진수로 인코딩 된 메시지를 가져 와서 cli rest 메시지와 함께 보낼 준비가 된 메시지``


# Examples

The following example focuses on using an utxo as input, the few differences when transfering from an account will be pointed out when necessary.
There is also a script [here](https://github.com/input-output-hk/jormungandr/blob/master/scripts/send-transaction) to send a transaction from a faucet account to a specific address which could be used as a reference.

``다음 예는 utxo를 입력으로 사용하는 데 중점을 두며 필요한 경우 계정에서 전송할 때의 몇 가지 차이점이 지적됩니다. 수도꼭지 계정에서 참조로 사용할 수있는 특정 주소로 트랜잭션을 전송하는 스크립트도 있습니다.``

Let's use the following utxo as input and transfer 50 lovelaces to the destination address

``다음 utxo를 입력으로 사용하고 목적지 주소로 50 lovelaces를 전송합시다``

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


# REST

Jormungandr comes with a CLI client for manual communication with nodes over HTTP.

## Conventions

Many CLI commands have common arguments:

- `-h <addr>` or `--host <addr>` - Node API address. Must always have `http://` or
`https://` prefix. E.g. `-h http://127.0.0.1`, `--host https://node.com:8443/cardano/api`
- `--debug` - Print additional debug information to stderr.
The output format is intentionally undocumented and unstable
- `--output-format <format>` - Format of output data. Possible values: json, yaml, default yaml.
Any other value is treated as a custom format using values from output data structure.
Syntax is Go text template: https://golang.org/pkg/text/template/.

## Node stats

Fetches node stats

```
jcli rest v0 node stats get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
# Number of blocks received by node
blockRecvCnt: 1102
# The Epoch and slot Number of the block (optional)
lastBlockDate: "20.29"
# Sum of all fee values in all transactions in last block
lastBlockFees: 534
# The block hash, it's unique identifier in the blockchain (optional)
lastBlockHash: b9597b45a402451540e6aabb58f2ee4d65c67953b338e04c52c00aa0886bd1f0
# The block number, in order, since the block0 (optional)
lastBlockHeight: 202901
# Sum of all input values in all transactions in last block
lastBlockSum: 51604
# When last block was created, not set if none was created yet (optional)
lastBlockTime: 2019-08-12T11:20:52.316544007+00:00
# Number of transactions in last block
lastBlockTx: 2
# State of the node
state: Running
# Number of transactions received by node
txRecvCnt: 5440
# Node uptime in seconds
uptime: 20032
```

## Whole UTXO

Fetches whole UTXO

```
jcli rest v0 utxo get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
- in_idx: 0                                                                 # input index
  in_txid: 50f21ac6bd3f57f231c4bf9c5fff7c45e2529c4dffed68f92410dbf7647541f1 # input transaction hash in hex
  out_addr: ca1qvqsyqcyq5rqwzqfpg9scrgwpugpzysnzs23v9ccrydpk8qarc0jqxuzx4s  # output address in bech32
  out_value: 999999999                                                      # output value
```

## Post transaction

Posts a signed, hex-encoded transaction

```
jcli rest v0 message post <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- -f --file <file_path> - File containing hex-encoded transaction.
If not provided, transaction will be read from stdin.


Fragment Id is printed on success (which can help finding transaction status using get message log command)

```
50f21ac6bd3f57f231c4bf9c5fff7c45e2529c4dffed68f92410dbf7647541f1
```

## Get message log

Get the node's logs on the message pool. This will provide information on pending transaction,
rejected transaction and or when a transaction has been added in a block

```
jcli rest v0 message logs <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)

YAML printed on success

```yaml
---
- fragment_id: 7db6f91f3c92c0aef7b3dd497e9ea275229d2ab4dba6a1b30ce6b32db9c9c3b2 # hex-encoded fragment ID
  last_updated_at: 	2019-06-02T16:20:26.201000000Z                              # RFC3339 timestamp of last fragment status change
  received_at: 2019-06-02T16:20:26.201000000Z                                   # RFC3339 timestamp of fragment receivement
  received_from: Network,                                                       # how fragment was received
  status: Pending,                                                              # fragment status
```

`received_from` can be one of:

```yaml
received_from: Rest     # fragment was received from node's REST API
```

```yaml
received_from: Network  # fragment was received from the network
```

`status` can be one of:

```yaml
status: Pending                 # fragment is pending
```

```yaml
status:
  Rejected:                     # fragment was rejected
    reason: reason of rejection # cause
```

```yaml
status:                         # fragment was included in a block
  InABlock: "6637.3"            # block epoch and slot ID formed as <epoch>.<slot_id>
```

## Blockchain tip

Retrieves a hex-encoded ID of the blockchain tip

```
jcli rest v0 tip get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)

## Get block

Retrieves a hex-encoded block with given ID

```
jcli rest v0 block <block_id> get <options>
```

<block_id> - hex-encoded block ID

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)

## Get next block ID

Retrieves a list of hex-encoded IDs of descendants of block with given ID.
Every list element is in separate line. The IDs are sorted from closest to farthest.

```
jcli rest v0 block <block_id> next-id get <options>
```

<block_id> - hex-encoded block ID

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- -c --count <count> - Maximum number of IDs, must be between 1 and 100, default 1

## Get account state

Get account state

```
jcli rest v0 account get <account-id> <options>
```

<account-id> - ID of an account, bech32-encoded

The options are
- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)

YAML printed on success

```yaml
---
counter: 1
delegation: c780f14f9782770014d8bcd514b1bc664653d15f73a7158254730c6e1aa9f356
value: 990
```

* `value` is the current balance of the account;
* `counter` is the number of transactions performed using this account
  this is useful to know when signing new transactions;
* `delegation` is the Stake Pool Identifier the account is delegating to.
  it is possible this value is not set if there is no delegation certificate
  sent associated to this account.

## Node settings

Fetches node settings

```
jcli rest v0 settings get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
block0Hash: 8d94ecfcc9a566f492e6335858db645691f628b012bed4ac2b1338b5690355a7  # block 0 hash of
block0Time: "2019-07-09T12:32:51+00:00"         # block 0 creation time of
consensusVersion: bft                           # currently used consensus
currSlotStartTime: "2019-07-09T12:55:11+00:00"  # current slot start time
fees:                                           # transaction fee configuration
  certificate: 4                                # fee per certificate
  coefficient: 1                                # fee per every input and output
  constant: 2                                   # fee per transaction
maxTxsPerBlock: 100                             # maximum number of transactions in block
```

## Node shutdown

Node shutdown

```
jcli rest v0 shutdown get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)

## Get leaders

Fetches list of leader IDs

```
jcli rest v0 leaders get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
- 1 # list of leader IDs
- 2
```

## Register leader

Register new leader and get its ID

```
jcli rest v0 leaders post <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)
-f, --file <file> - File containing YAML with leader secret. It must have the same format as secret YAML passed to Jormungandr as --secret. If not provided, YAML will be read from stdin.

On success created leader ID is printed

```
3
```

## Delete leader

Delete leader with given ID

```
jcli rest v0 leaders delete <id> <options>
```

<id> - ID of deleted leader

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)

## Get leadership logs

Fetches leadership logs

```
jcli rest v0 leaders logs get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
- created_at_time: "2019-08-19T12:25:00.417263555+00:00"
  enclave_leader_id: 1
  finished_at_time: "2019-08-19T23:19:05.010113333+00:00"
  scheduled_at_date: "0.3923"
  scheduled_at_time: "2019-08-19T23:18:35+00:00"
  wake_at_time: "2019-08-19T23:18:35.001254555+00:00"
```

## Get stake pools

Fetches list of stake pool IDs

```
jcli rest v0 stake-pools get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
- 5cf03f333f37eb7b987dbc9017b8a928287a3d77d086cd93cd9ad05bcba7e60f # list of stake pool IDs
- 3815602c096fcbb91072f419c296c3dfe1f730e0f446a9bd2553145688e75615
```

## Get stake distribution

Fetches stake information

```
jcli rest v0 stake get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
epoch: 228      # Epoch of last block
stake:
  dangling: 0 # Total value stored in accounts, but assigned to nonexistent pools
  pools:
    - - 5cf03f333f37eb7b987dbc9017b8a928287a3d77d086cd93cd9ad05bcba7e60f # stake pool ID
      - 1000000000000                                                    # staked value
    - - 3815602c096fcbb91072f419c296c3dfe1f730e0f446a9bd2553145688e75615 # stake pool ID
      - 1000000000000                                                    # staked value
  unassigned: 0 # Total value stored in accounts, but not assigned to any pool
```

## Network stats

Fetches network stats

```
jcli rest v0 network stats get <options>
```

The options are

- -h <node_addr> - see [conventions](#conventions)
- --debug - see [conventions](#conventions)
- --output-format <format> - see [conventions](#conventions)


YAML printed on success

```yaml
---
  # hex-encoded node ID
- nodeId: 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20
  # timestamp of when the connection was established
  establishedAt: "2019-10-14T06:24:12.010231281+00:00"
  # timestamp of last time block was received from node if ever (optional)
  lastBlockReceived: "2019-10-14T00:45:57.419496113+00:00"
  # timestamp of last time fragment was received from node if ever (optional)
  lastFragmentReceived: "2019-10-14T00:45:58.419496150+00:00"
  # timestamp of last time gossip was received from node if ever (optional)
  lastGossipReceived: "2019-10-14T00:45:59.419496188+00:00"
```
