# jcli

This is the node command line helper. It is mostly meant for developers and
stake pool operators. It allows offline operations:

* generating cryptographic materials for the wallets and stake pools;
* creating addresses, transactions and certificates;
* prepare a new blockchain

``jcli는 노드 명령 행 헬퍼입니다. 주로 개발자와 스테이크 풀 운영자를 위한 것입니다. 오프라인 작업을 허용합니다.``

* ``지갑 및 스테이크 풀을 위한 암호화 자료를 생성하는 단계``
* ``주소, 거래, 인증서 작성``
* ``새로운 블록체인 준비``

and it allows simple interactions with the node:

* query stats;
* send transactions and certificates;
* get raw blocks and UTxOs.

``노드와의 간단한 상호 작용을 허용합니다.``

* ``쿼리 통계``
* ``거래 및 인증서 보내기``
* ``raw blocks 및 UTxO 얻기``

# cryptographic keys

There are multiple type of key for multiple use cases.

``여러 사용 사례에 대한 다양한 유형의 키가 있습니다.``

| 유형 | 사용법 |
|:----:|:------|
|`Ed25519` | 알고리즘의 서명 알고리즘 |
|`Ed25519Bip32`| HDWallet 관련, 체인코드로 확장된 HDWallet |
|`Ed25519Extended`| 체인코드 없이 Ed25519Bip32와 관련됨 |
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

``개인 키로 데이터에 서명하십시오. 지원되는 키 형식은 ed25519, ed25519bip32, ed25519extended, sumed25519_12입니다.``

```
jcli key sign <options> <data>
```

The options are

- --secret-key <secret_key> : bech32로 인코딩 된 비밀 키가 있는 파일 경로

- -o, --output <output> : 서명을 쓸 파일의 경로입니다. 값이 전달되지 않으면 표준 출력이 사용됩니다.



``<data> -서명 할 데이터가 있는 파일 경로, 값이 전달되지 않으면 표준 입력이 사용됩니다.``


## Verifying signed data

Verify signed data with public key. Supported key formats are: ed25519, ed25519bip32 and
sumed25519_12.

``공개 키로 서명 된 데이터를 확인하십시오. 지원되는 키 형식은 ed25519, ed25519bip32, sumed25519_12입니다.``

```
jcli key verify <options> <data>
```

The options are
- --public-key <public_key> : bech32로 인코딩 된 공개 키가있는 파일 경로
- --signature <signature> : 서명이있는 파일의 경로


``- <data> 서명 할 데이터가 있는 파일 경로, 값이 전달되지 않으면 표준 입력이 사용됩니다.``


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

``다음의 각 명령을 사용하면 배포용 블록체인 또는 테스트 블록체인에 대한 주소를 만들 수 있습니다. 테스트를 하는 경우 --testing 플래그를 사용해야합니다.``

There's 3 types of addresses:

* Single address : A simple spending key. This doesn't have any stake in the system
* Grouped address : A spending key attached to an account key. The stake is automatically
* Account address : An account key. The account is its own stake

``3 가지 유형의 주소가 있습니다.``

* ``Single address : 간단한 지출 키. 이 시스템에 아무런 지분이 없습니다``

* ``Grouped address : account 키에 연결된 지출 키. 스테이크는 자동으로``

* ``Account address : account 키. account 자체가 지분입니다``


### Address for UTxO

You can create a single address (non-staked) using the spending public key for
this address utilising the following command:

``다음 명령에서 지출 공개 키를 사용하여 단일 주소(스테이크되지 않는)를 만들 수 있습니다.``

```
$ jcli address \
    single ed25519e_pk1jnlhwdgzv3c9frknyv7twsv82su26qm30yfpdmvkzyjsdgw80mfqduaean
ca1qw207ae4qfj8q4yw6v3ned6psa2r3tgrw9u3y9hdjcgj2p4pcaldyukyka8
```

To add the staking information and make a group address, simply add the account
public key as a second parameter of the command:

``스테이킹 정보를 추가하고 그룹 주소를 만들려면 account 공개 키를 명령의 두 번째 매개 변수로 추가하십시오.``

```
$ jcli address \
    single \
    ed25519_pk1fxvudq6j7mfxvgk986t5f3f258sdtw89v4n3kr0fm6mpe4apxl4q0vhp3k \
    ed25519_pk1as03wxmy2426ceh8nurplvjmauwpwlcz7ycwj7xtl9gmx9u5gkqscc5ylx
ca1q3yen35r2tmdye3zc5lfw3x992s7p4dcu4jkwxcda80tv8xh5ym74mqlzudkg42443nw08cxr7e9hmcuzals9ufsa9uvh723kvteg3vpvrcxcq
```

### Address for Account

To create an account address you need the account public key and run:

``account 주소를 만들려면 account 공개 키가 필요합니다.``

```
$ jcli address \
    account ed25519_pk1c4yq3hflulynn8fef0hdq92579n3c49qxljasrl9dnuvcksk84gs9sqvc2
ca1qhz5szxa8lnujwva8997a5q42nckw8z55qm7tkq0u4k03nz6zc74ze780qe
```

### changing the address prefix

You can decide to change the address prefix, allowing you to provide more
enriched data to the user. However, this prefix is not forwarded to the node,
it is only for UI/UX.

``주소 접두사를 변경하여 보다 풍부한 데이터를 사용자에게 제공 할 수 있습니다. 그러나 이 접두부는 노드로 전달되지 않으며 UI/UX 전용입니다.``

```
$ jcli address \
    account \
    --prefix=address_ \
    ed25519_pk1yx6q8rsndawfx8hjzwntfs2h2c37v5g6edv67hmcxvrmxfjdz9wqeejchg
address_1q5smgquwzdh4eyc77gf6ddxp2atz8ej3rt94nt6l0qes0vexf5g4cw68kdx
```


# transaction

Tooling for offline transaction creation and signing.

``오프라인 트랜잭션 생성 및 서명을 위한 명령어.``

```
jcli transaction
```

Those familiar with [`cardano-cli`](http://github.com/input-output-hk/cardano-cli)
transaction builder will see resemblance in `jcli transaction`.

``cardano-cli transaction builder에 익숙한 사람들은 jcli transaction과 유사합니다.``

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



``다음과 같은 명령을 사용할 수 있습니다.``

1. ``거래 준비`` :
    - `` `new` 비어있는 새 트랜잭션을 새로 작성하십시오;``
    - `add-input`
    - `add-account`
    - `add-output`


2. `` `finalize` : 서명을 위한 트랜잭션 마무리합니다:``


3. ``증인을 만들고 증인을 추가하십시오:``
    - ``make-witness``
    - ``add-witness``


4. ``'seal' 거래를 봉인하고 블록 체인에 보낼 준비가 되었습니다.``


There are also functions to help decode and display the
content information of a transaction:

* `info`
* `id` to get the **Transaction ID** of the transaction
* `to-message` to get the hexadecimal encoded message, ready to send with `cli rest message`



``트랜잭션의 컨텐츠 정보를 디코딩하고 표시하는 데 도움이되는 기능도 있습니다.``

* ``'info'``

* ``'id' 거래 ID를 얻기 위한 것``

* ``'to-message' 16진수로 인코딩 된 메시지를 가져 와서 'cli rest message' 와 함께 보낼 준비를 하는 것``


# Examples

The following example focuses on using an utxo as input, the few differences when transfering from an account will be pointed out when necessary.
There is also a script [here](https://github.com/input-output-hk/jormungandr/blob/master/scripts/send-transaction) to send a transaction from a faucet account to a specific address which could be used as a reference.

``다음 예는 utxo를 입력으로 사용하는 데 중점을 두며 account에서 전송하는 경우와 몇 가지 차이점을 설명합니다.``

``facuet 에서 특정 주소로 트랜잭션을 보내는 스크립트도 있습니다. 참조로 사용됩니다.``

Let's use the following utxo as input and transfer 50 lovelaces to the destination address

``다음 utxo를 입력으로 사용하고 목적지 주소로 50 lovelaces 를 전송합시다``

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

``입력을 위해, 우리는 UTXO's transaction ID 및 UTXO'S output index 등의 UTXO 필드 참조가 필요 하며 associated value 필드와 함께 얼마나 많은 코인이 있는지 지정해야 합니다.``

### Example - UTXO address as Input

```sh
jcli transaction add-input  55762218e5737603e6d27d36c8aacf8fcd16406e820361a8ac65c7dc663f6d1c 0 100 --staging tx
```

### Example - Account address as Input

If the input is an account, the command is slightly different

``입력이 account 인 경우 명령이 약간 다릅니다``

```sh
jcli transaction add-account account_address account_funds --staging tx
```

## Add output

For the output, we need the address we want to transfer to, and the amount.

``출력을 위해, 전송할 주소와 금액이 필요합니다.``

```sh
jcli transaction add-output  ca1qvnr5pvt9e5p009strshxndrsx5etcentslp2rwj6csm8sfk24a2wlqtdj6 50 --staging tx
```

## Add fee and change address

We want to get the change in the same address that we are sending from (the *associated address* of the utxo). We also specify how to compute the fees.
You can leave out the `--fee-constant 5 --fee-coefficient 2` part if those are both 0.

``보내는 주소와 동일한 주소(utxo 관련 주소)를 변경하려고 합니다. 또한 수수료 계산 방법을 지정합니다. --fee-constant 5 --fee-coefficient 2 로 설정합니다. 이 2개 옵션이 0 인 경우 생략 할 수 있습니다.``


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

``트랜잭션에 서명하려면 입력 주소(utxos에 있는 주소)의 개인 키와 제네시스 블록 해시가 필요합니다.``

The genesis' hash is needed for ensuring that the transaction cannot be re-used in another blockchain and for security concerns on offline transaction signing, as we are signing the transaction for the specific blockchain started by this block0 hash.

``우리가 이 block0 해시에 의해 시작된 특정 블록체인에 대한 거래에 서명 할 때, 트랜잭션을 다른 블록 체인에서 재사용 할 수 없도록 하고 오프라인 거래 서명에 대한 보안 문제를 위해 제네시스의 해시가 필요합니다.``

The following command takes the private key in the *key.prv* file and creates a witness in a file named *witness* in the current directory. 

``다음 명령은 key.prv 파일에서 개인 키를 가져 와서 현재 디렉토리의 witness 파일에 감시를 만듭니다.``

```sh
jcli transaction make-witness --genesis-block-hash abcdef987654321... --type utxo txid witness key.prv
```

---
#### Account input

When using an account as input, the command takes `account` as the type and an additional parameter: `--account-spending-counter`, that should be increased every time the account is used as input.

``account를 입력으로 사용하는 경우 이 명령은 입력으로 사용할 때마다 증가해야 하는 --account-spending-counter 유형 및 추가 매개 변수를 고려합니다.``

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

``REST API를 사용하여 보내십시오.``

```sh
jcli rest v0 message post -f txmsg --host http://127.0.0.1:8443/api
```

## Checking if the transaction was accepted

You can check if the transaction was accepted by checking the node logs, for example, if the transaction is accepted 

``트랜잭션이 승인되면 노드 로그를 확인하여 트랜잭션이 승인 여부를 확인할 수 있습니다.``

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

``InABlock 상태는 날짜가 "4.707"이고 d9040ca57e513a36ecd3bb54207dfcd10682200929cad6ada46b521417964174 블록에서 트랜잭션이 수락되었음을 의미합니다.``

The status here could also be: 

``여기서 status는 아래에서 설명합니다.``

Pending: if the transaction is received and is pending being added in the blockchain (or rejected).

``Pending : 거래가 접수되어 블록체인에 추가(또는 거부) 중인 경우.``

or

Rejected: with an attached message of the reason the transaction was rejected. 

``Rejected : 거래가 거부 된 이유에 대한 메시지가 첨부되어 있습니다.``


# certificate

Tooling for offline transaction creation

``오프라인 트랜잭션 생성을 위한 명령어``

## Builder

Builds a signed certificate.

``서명된 인증서를 작성합니다.``

The process can be split into steps on the first step certificate
is created.

``인증서 작성의 첫 번째 단계는 인증서 작성 단계 입니다.``

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

``출력 파일이 생략되면 결과가 stdout에 기록됩니다. 인증서가 준비되면 소유자의 개인 키로 서명해야합니다.``


```
jcli certificate sign <key> <input-file> <output-file>
```



# Genesis

Tooling for working with a genesis file

``제네시스 파일 작업을 위한 명령어``

# Usage
```sh
jcli genesis [subcommand]
```

## Subcommands

- decode: ``인코딩 된 제네시스 블록에 해당하는 YAML 파일 인쇄 하기``
- encode: ``주어진 yaml 파일에서 블록체인의 제네시스 블록 생성 하기.``
- hash: ``제네시스 블록 해시 인쇄 하기`` 
- init: ``YAML 파일 작성에 도움이 되는 적절한 문서를 사용하여 기본 제네시스 파일 생성 하기.``
- help



## Examples

### Encode a genesis file

```sh
jcli genesis encode --input genesis.yaml --output block-0.bin
```

or equivantely

``또는 동등하게``

```sh
cat genesis.yaml | jcli genesis encode > block-0.bin
```

### Get the hash of an encoded genesis file

```sh
jcli genesis hash --input block-0.bin
```








# REST

Jormungandr comes with a CLI client for manual communication with nodes over HTTP.

``Jormungandr 에는 HTTP를 통한 노드와의 수동 통신을 위한 CLI 클라이언트가 제공됩니다.``

## Conventions

Many CLI commands have common arguments:

``많은 CLI 명령에는 일반적인 인수가 있습니다.``

- `-h <addr> or --host <addr>` : Node API 주소. 항상 `http://` 또는 `https://` 접두사가 있어야 합니다. 예 : `-h http://127.0.0.1`, `--host https://node.com:8443/cardano/api`
- `--debug` : 추가 디버그 정보를 stderr에 인쇄합니다. 출력 형식이 의도적으로 문서화되지 않고 불안정합니다.
- `--output-format <format>` : 출력 데이터의 형식입니다. 가능한 값 : json, yaml, default yaml. 다른 값은 출력 데이터 구조의 값을 사용하여 사용자 정의 형식으로 처리됩니다. 구문은 Go 텍스트 템플릿입니다 (https://golang.org/pkg/text/template/).



## Node stats

Fetches node stats

``노드 통계를 가져옵니다.``

```
jcli rest v0 node stats get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크  [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)



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

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크  [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)


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

``서명 된 16진 인코딩 트랜잭션을 게시합니다.``

```
jcli rest v0 message post <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- -f --file <file_path> : 16진으로 인코딩 된 트랜잭션을 포함하는 파일입니다. 제공하지 않으면 stdin에서 트랜잭션을 읽습니다.


Fragment Id is printed on success (which can help finding transaction status using get message log command)

``프래그먼트 ID는 성공시 인쇄됩니다 (get message log 명령을 사용하여 트랜잭션 상태를 찾는 데 도움이 됨)``

```
50f21ac6bd3f57f231c4bf9c5fff7c45e2529c4dffed68f92410dbf7647541f1
```

## Get message log

Get the node's logs on the message pool. This will provide information on pending transaction,
rejected transaction and or when a transaction has been added in a block

``메시지 풀에서 노드의 로그를 가져옵니다. 보류중인 거래, 거부 된 거래, 거래가 블록에 추가 된 시기에 대한 정보를 제공합니다.``

```
jcli rest v0 message logs <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)
    

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

``블록체인 tip의 16진 인코딩 ID를 검색합니다.``

```
jcli rest v0 tip get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug - 참조링크 [conventions](#conventions)



## Get block

Retrieves a hex-encoded block with given ID

``주어진 ID로 16진으로 인코딩 된 블록을 검색합니다.``

```
jcli rest v0 block <block_id> get <options>
```

<block_id> - hex-encoded block ID

``<block_id> - 16진수로 인코딩 된 블록 ID``


The options are

- -h <node_addr> - 참조링크 [conventions](#conventions)
- --debug - 참조링크 [conventions](#conventions)



## Get next block ID

Retrieves a list of hex-encoded IDs of descendants of block with given ID.
Every list element is in separate line. The IDs are sorted from closest to farthest.

``주어진 ID를 가진 블록 자손의 16진 인코딩 된 ID 목록을 검색합니다. 모든 목록 요소는 별도의 줄에 있습니다. ID는 가장 가까운 것부터 가장 먼 것까지 정렬됩니다.``

```
jcli rest v0 block <block_id> next-id get <options>
```

<block_id> - hex-encoded block ID

``<block_id> - 16진수로 인코딩 된 블록 ID``

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- -c --count <count> : 최대 ID 수는 1에서 100 사이 여야하며 기본값은 1입니다.



## Get account state

Get account state

``account 상태 얻기``

```
jcli rest v0 account get <account-id> <options>
```

<account-id> - ID of an account, bech32-encoded

``<account-id> : bech32로 인코딩 된 account의 ID``

The options are
- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)


YAML printed on success

```yaml
---
counter: 1
delegation: c780f14f9782770014d8bcd514b1bc664653d15f73a7158254730c6e1aa9f356
value: 990
```

* `value` 는 account의 현재 잔고입니다.;
* `counter` 는 이 account를 사용하여 수행 된 트랜잭션 수이며 새 트랜잭션에 서명 할 때 알아야합니다.;
* `delegation` 은 account가 위임 된 스테이크 풀 식별자입니다. 이 account와 관련된 위임 인증서가 전송되지 않은 경우 이 값이 설정되지 않을 수 있습니다.


## Node settings

Fetches node settings

``노드 설정을 가져옵니다.``

```
jcli rest v0 settings get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)


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

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)

## Get leaders

Fetches list of leader IDs

``리더 ID 목록을 가져옵니다.``

```
jcli rest v0 leaders get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)


YAML printed on success

```yaml
---
- 1 # list of leader IDs
- 2
```

## Register leader

Register new leader and get its ID

``새로운 리더를 등록하고 ID를 얻습니다``

```
jcli rest v0 leaders post <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)
-f, --file <file> : leader secret 이 포함 된 YAML 파일. secrey YAML이 Jormungandr에 --secret과 같은 형식으로 전달되어야합니다. 제공하지 않으면 stdin에서 YAML을 읽습니다.


On success created leader ID is printed

``성공하면 리더 ID가 인쇄됩니다.``

```
3
```

## Delete leader

Delete leader with given ID

`` 주어진 아이디로 리더 삭제 ``

```
jcli rest v0 leaders delete <id> <options>
```

<id> - ID of deleted leader

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)



## Get leadership logs

Fetches leadership logs

```
jcli rest v0 leaders logs get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)



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

`` 스테이크 풀 ID 리스트 가져오기 ``

```
jcli rest v0 stake-pools get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)


YAML printed on success

```yaml
---
- 5cf03f333f37eb7b987dbc9017b8a928287a3d77d086cd93cd9ad05bcba7e60f # list of stake pool IDs
- 3815602c096fcbb91072f419c296c3dfe1f730e0f446a9bd2553145688e75615
```

## Get stake distribution

Fetches stake information

`` 스테이크 정보 가져오기``

```
jcli rest v0 stake get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)



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

`` 네트워크 통계 가져요기``

```
jcli rest v0 network stats get <options>
```

The options are

- -h <node_addr> : 참조링크 [conventions](#conventions)
- --debug : 참조링크 [conventions](#conventions)
- --output-format <format> : 참조링크 [conventions](#conventions)



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
