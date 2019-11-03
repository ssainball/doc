# Quickstart

The rust node comes with tools and help in order to quickly start
a node and connect to the blockchain.

``RUST 노드는 신속하게 노드를 시작하고 블록체인에 연결하기 위한 도구와 도움말이 제공됩니다.``

It is compatible with most platforms and it is pre-packaged for some
of them.

``대부분의 플랫폼과 호환되며 일부 플랫폼용으로 미리 패키징 되어 있습니다.``

Here we will see how to install `jormungandr` and its helper `jcli`
and how to connect quickly to a given blockchain.

``여기에서 jormungandr와 그 도우미 jcli를 설치하는 방법과 주어진 블록체인에 빠르게 연결하는 방법을 알아 볼 것입니다.``

There are three posible ways you can start jormungandr.

``jormungandr를 시작할 수 있는 세 가지 방법이 있습니다.``

## As a passive node in an existing network

As described [here](./node_types/01_passive_node.md).  

The passive Node is the most common type of Node on the network. It can be used to download the blocks and broadcast transactions to peers, but it
doesn't have cryptographic materials or any mean to create blocks.
This type of nodes are mostly used for wallets, explorers or relays.

``수동 노드는 네트워크에서 가장 일반적인 유형의 노드입니다. 블록을 다운로드하고 피어에게 트랜잭션을 브로드캐스트하는 데 사용할 수 있지만 암호화 자료나 블록을 만들 수단이 없습니다. 이 유형의 노드는 주로 지갑, 탐색기 또는 릴레이에 사용됩니다.``

## As a node generating blocks in an existing network

The network could be running either bft or genesis consensus. In the former case the node must have the private key of a registered as a slot leader, while for the latter the private keys of a registered stake pool are needed. 

``네트워크는 bft 또는 제네시스 합의를 실행할 수 있습니다. 전자의 경우 노드에는 슬롯 리더로 등록 된 개인 키가 있어야하고 후자에는 등록 된 스테이크 풀의 개인 키가 필요합니다.``

More information [here](./node_types/02_generating_blocks.md)

## Creating your own network

This is similar to the previous case, but configuring a genesis file is needed. Consult the [Advanced section](../advanced/introduction.md) for more information on this procedure.

``이것은 이전 경우와 비슷하지만 제네시스 파일을 구성해야합니다. 이 절차에 대한 자세한 내용은 고급 섹션을 참조하십시오.``

The software is bundled with 2 different command line software:

``이 소프트웨어는 두 가지 명령 줄 소프트웨어와 함께 번들로 제공됩니다.``

1. **jormungandr**: the node;
2. **jcli**: Jormungandr Command Line Interface, the helpers and primitives to run and interact with the node.

# Installation

## From a release

This is the recommended method. Releases are all available
[here](https://github.com/input-output-hk/jormungandr/releases).

``이것이 권장되는 방법입니다. 릴리스는 모두 여기에서 사용할 수 있습니다.``

## From source

Jormungandr's code source is available on
[github](https://github.com/input-output-hk/jormungandr#how-to-install-from-sources).
Follow the instructions to build the software from sources.

``Jormungandr의 코드 소스는 github에서 사용할 수 있습니다. 지침에 따라 소스에서 소프트웨어를 빌드하십시오.``

# Help and auto completion

All commands come with usage help with the option `--help` or `-h`.

``모든 명령에는 --help 또는 -h 옵션과 함께 사용 도움말이 제공됩니다.``

For `jcli`, it is possible to generate the auto completion with:

``jcli의 경우 다음을 사용하여 자동 완성을 생성 할 수 있습니다.``

```
jcli auto-completion bash ${HOME}/.bash_completion.d
```

Supported shells are: bash, fish, zsh, powershell and elvish.

``지원되는 쉘은 bash, fish, zsh, powershell, elvish입니다.``

**Note:** 
Make sure `${HOME}/.bash_completion.d` directory previously exists on your HD.
In order to use auto completion you still need to:

``참고 : ${HOME}/.bash_completion.d 디렉토리가 HD에 이미 존재하는지 확인하십시오. 자동 완성을 사용하려면 다음을 수행해야합니다.``

```
source ${HOME}/.bash_completion.d/jcli.bash
```
You can also put it in your `${HOME}/.bashrc`.

''${HOME}/.bashrc에 넣을 수도 있습니다.``



In order to start the node, you first need to gather the blockchain
information you need to connect to.

``노드를 시작하려면 먼저 연결해야하는 블록체인 정보를 수집해야 합니다.``

1. the hash of the **genesis block** , 이것은 블록체인 신뢰의 근원입니다. 64개의 16진수 입니다.
2. the **trusted peers** 식별자 및 and access points.

These information are essentials to start your node in a secure way.

``이 정보는 노드를 안전하게 시작하는 데 필수적입니다.``

The **genesis block** is the first block of the blockchain. It contains the
static parameters of the blockchain as well as the initial funds. Your node
will utilise the **Hash** to retrieve it from the other peers. It will also
allows the Node to verify the integrity of the downloaded **genesis block**.

``기원 블록은 블록체인의 첫 번째 블록입니다. 초기 자금규모뿐만 아니라 블록체인의 정적 매개 변수가 포함되어 있습니다. 노드는 해시를 사용하여 다른 피어에서 검색합니다. 또한 노드가 다운로드된 기원 블록의 무결성을 확인할 수 있습니다.``

The **trusted peers** are the nodes in the public network that your Node will
trust in order to initialise the Peer To Peer network.

`` trusede peers 는 당신의 노드를 초기화하기 위해 퍼블릭 네트워크에서 믿을 수 있는 노드입니다.``

# The node configuration

Your node configuration file may look like the following:

``노드 구성 파일은 다음과 같습니다.``

**Note**

This config shouldn't work as it is, the ip address and port for the trusted peer should be those of an already running node.
Also, the public_address ('u.x.v.t') should be a valid address (you can use an internal one, eg: 127.0.0.1).
Furthermore, you need to have permission to write in the path specified by the storage config.

``이 구성은 그대로 작동하지 않아야합니다. 트러스트된 피어의 IP 주소 및 포트는 이미 실행중인 노드의 IP 주소 및 포트 여야합니다. 또한 public_address('u.x.v.t')는 유효한 주소 여야합니다(예 : 127.0.0.1). 또한 스토리지 구성에 지정된 경로에 쓸 수 있는 권한이 있어야 합니다.``

```yaml
storage: "/mnt/cardano/storage"

rest:
  listen: "127.0.0.1:8443"

p2p:
  trusted_peers:
    - address: "/ip4/104.24.28.11/tcp/8299"
      id: ed25519_pk14uednfkcqq2u2w4uhy7saahyqapn78huu8fj7tv84rqeer78hcvs8h6r6u
```

Description of the fields:

- `storage`: (선택사항) 저장소 경로입니다. 생략하면 블록체인은 메모리에만 저장됩니다.
- `log`: (optional) Logging configuration:
    - `level`: log messages minimum severity. 설정하지 않으면 기본 값은 "info" 입니다.
        Possible values: "off", "critical", "error", "warn", "info", "debug", "trace".
    - `format`: Log output format, `plain` or `json`.
    - `output`: 로그 출력 대상. 가능한 값은 다음과 같습니다.:
      - `stdout`: standard output
      - `stderr`: standard error
      - `syslog`: syslog (only available on Unix systems)
      - `journald`: journald service (only available on Linux with systemd,
        (if jormungandr is built with the `systemd` feature)
      - `gelf`: Configuration fields for GELF (Graylog) network logging protocol
        (if jormungandr is built with the `gelf` feature):
        - `backend`: _hostname_:_port_ of a GELF server
        - `log_id`: identifier of the source of the log, for the `host` field
                    in the messages.
- `rest`: (optional) Configuration of the REST endpoint.
    - `listen`: _address_:_port_ to listen for requests
    - `pkcs12`: (optional) Certificate file
    - `cors`: (optional) CORS configuration, if not provided, CORS is disabled
      - `allowed_origins`: (선택 사항) 허용 된 원점 (제공된 경우)
      - `max_age_secs`: (선택 사항) 최대 CORS 캐싱 시간 (초) (제공된 것이 없으면 캐싱이 비활성화 됨)
- `p2p`: P2P network settings
    - `trusted_peers`: (선택 사항) P2P 토폴로지와 로컬 블록체인을 부트스트랩하기 위해 연결할 multiaddr(노드 public_id) 목록입니다.;
    - `private_id`: 네트워크에서 이 노드를 식별하는 데 사용될 노드의 개인 키 (Ed25519)
    - `public_address`: P2P 서비스의 주소 지정. 이것은 노드의 블록체인 보급에 관심이 있는 네트워크의 다른 피어에게 배포 될 공개 주소입니다.
    - `listen_address`: (선택 사항) multiaddr은 p2p 연결을 수신하기 위해 노드가 수신 할 주소를 지정합니다. 비워 둘 수 있으며 노드는 public_address에 지정된 값을 수신합니다.
    - `topics_of_interest`: The dissemination topics this node is interested to hear about:
      - `messages`: 거래 및 기타 원장 항목.
        non-mining node: `low`.  stakepool: `high`; 
      - `blocks`: 새로운 블록에 대한 알림.
        non-mining node: `normal`. stakepool: `high`.
    - `max_connections`: 이 노드가 유지해야하는 최대 동시 P2P 연결 수입니다.
- `explorer`: (optional) Explorer settings
    - `enabled`: True or false

[multiaddr]: https://github.com/multiformats/multiaddr

# Starting the node

```
jormungandr --config config.yaml --genesis-block-hash 'abcdef987654321....'
```

The 'abcdef987654321....' part refers to the hash of the genesis, that should be given to you from one of the peers in the network you are connecting to.

``'abcdef987654321 ....' 부분은 연결중인 네트워크의 피어 중 하나에서 제공해야 하는 기원의 해시를 나타냅니다.``

In case you have the genesis file (for example, because you are creating the network) you can get this hash with jcli.

``당신이 제네시스 파일을 가지고 있는 경우 (예를 들어, 당신이 네트워크를 만들고 있기 때문에) jcli를 사용하여 이 해시를 얻을 수 있습니다.``

```sh
cat block-0 | jcli genesis hash
```

or, in case you only have the yaml file

``또는 yaml 파일만 가지고 있는 경우``

```sh
cat genesis.yaml | jcli genesis encode | jcli genesis hash
```

[`Ed25519`]: ../jcli/key.md



It is possible to query the node via its REST Interface.

``REST 인터페이스를 통해 노드를 쿼리할 수 있습니다.``

In the node configuration, you have set something like:

``노드 구성에서 다음과 같이 설정``


```yaml
# ...

rest:
  listen: "127.0.0.1:8443"

#...
```

This is the REST endpoint to talk to the node, to query blocks or send transaction.

``노드와 통신하거나 블록을 쿼리하거나 트랜잭션을 보내는 REST 엔드 포인트입니다.``

It is possible to query the node stats with the following end point:

``다음과 같은 엔드 포인트로 노드 통계를 쿼리할 수 있습니다.``


```
curl http://127.0.0.1:8443/api/v0/node/stats
```

The result may be:

```json
{"blockRecvCnt":120,"txRecvCnt":92,"uptime":245}
```

> THE REST API IS STILL UNDER DEVELOPMENT

``REST API는 아직 개발 중입니다``

Please note that the end points and the results may change in the future.

``접속지점(end points) 및 결과는 향후 변경 될 수 있습니다.``

To see the whole Node API documentation,
[click here](https://editor.swagger.io/?url=https://raw.githubusercontent.com/input-output-hk/jormungandr/master/doc/openapi.yaml)

``전체 노드 API 설명서를 보려면 여기를 클릭하십시오``

# Explorer mode

The node can be configured to work as a explorer. This consumes more resources, but makes it possible to query data otherwise not available.

``탐색기로 작동하도록 노드를 구성할 수 있습니다. 이로 인해 더 많은 리소스가 소비되지만 접근 불가했던 감춰진 데이터를 쿼리할 수 있습니다.``

## Configuration

There is two ways of enabling the explorer api. It can either be done by passing the `--enable-explorer` flag on the start arguemnts or by the config file: 

``탐색기 API를 활성화하는 방법에는 두 가지가 있습니다. 시작 인수에 --enable-explorer 플래그를 전달하거나 구성 파일을 사용하여 수행할 수 있습니다.``

``` yaml
explorer:
    enabled: true
```

#### CORS

For configuring CORS the explorer API, this needs to be done on the REST section of the config, as documented [here](../configuration/network.md).

``CORS 탐색기 API를 구성하려면 여기에 설명된 대로 환경설정의 REST 섹션에서 수행해야 합니다.``

## API

A graphql interface can be used to query the explorer data, when enabled, two endpoints are available in the [REST interface](03_rest_api.md): `/explorer/graphql` and `/explorer/graphiql` .

``graphql 인터페이스를 사용하여 탐색기 데이터를 조회 할 수 있습니다. 사용 가능한 경우 REST 인터페이스에서 /explorer/graphql 및 /explorer /graphiql 의 두 엔드 포인트를 사용할 수 있습니다.``

The first is the one that queries are made against, for example: 

``첫 번째는 다음과 같이 쿼리하는 것입니다.``

``` sh
curl \
    -X POST \
    -H "Content-Type: application/json" \
    --data '{'\
        '"query": "{'\
        '   status {'\
        '       latestBlock {'\
        '           chainLength'\
        '           id'\
        '           previousBlock {'\
        '               id'\
        '           }'\
        '       }'\
        '   }'\
        '}"'\
    '}' \
  http://127.0.0.1:8443/explorer/graphql
```

While the second serves an in-browser graphql IDE that can be used to try queries interactively.
``두 번째는 대화식으로 쿼리를 시도하는 데 사용할 수 있는 브라우저 내 graphql IDE 를 제공합니다.``


# How to start a node as a leader candidate

## Gathering data

Like in the passive node case, two things are needed to connect to an existing network

``패시브 노드 사례처럼 기존 네트워크에 연결하려면 두 가지가 필요합니다.``

1. the hash of the **genesis block** , 이것은 블록체인 신뢰의 근원입니다. 64개의 16진수 입니다.
2. the **trusted peers** 식별자 및 and access points.



The node configuration could be the same as that for [running a passive node](./01_passive_node.md). 

``노드 구성은 수동 노드를 실행하는 것과 동일 할 수 있습니다.``

There are some differences depending if you are connecting to a network running a genesis or bft consensus protocol.

``당신이 독창적인 합의 프로토콜 또는 bft 합의 프로토콜을 실행하는 네트워크에 접속하는지에 따라 약간의 차이가 있습니다.``

### Connecting to a genesis blockchain

#### Registering a stake pool

In order to be able to generate blocks in an existing genesis network, a [registered stake pool](../../stake_pool/registering_stake_pool) is needed.

``기존 제네시스 네트워크에서 블록을 생성할 수 있으려면 등록된 스테이크 풀이 필요합니다.``

#### Creating the secrets file

Put the node id and private keys in a yaml file in the following way:  

``다음과 같은 방법으로 노드 ID 및 개인 키를 yaml 파일에 넣습니다.``

##### Example

filename: _node_secret.yaml_

```yaml
genesis:
  sig_key: Content of stake_pool_kes.prv file
  vrf_key: Content of stake_pool_vrf.prv file
  node_id: Content of stake_pool.id file
```

#### Starting the node 

```sh
jormungandr --genesis-block-hash asdf1234... --config config.yaml --secret node_secret.yaml
```

_The 'asdf1234...' part should be the actual block0 hash of the network_

``'asdf1234 ...'부분은 네트워크의 실제 block0 해시 여야합니다.``

### Connecting to a BFT blockchain

In order to generate blocks, the node should be registered as a slot leader in the network and started in the following way.

``블록을 생성하려면 노드를 네트워크에서 슬롯 리더로 등록하고 다음과 같이 시작해야합니다.``

## The secret file

Put secret key in a yaml file, e.g. `node_secret.yaml` as follows:

``다음과 같이 yaml 파일에 비밀키(예 : node_secret.yaml)를 넣으십시오. :``

```yaml
bft:
 signing_key: ed25519_sk1kppercsk06k03yk4qgea....
```

where signing_key is a private key associated to the public id of a slot leader.

``여기서 signing_key는 슬롯 리더의 공용 ID와 관련된 개인 키.``

### Starting the node

```sh
jormungandr --genesis-block asdf1234... --config node.config --secret node_secret.yaml
```

_The 'asdf1234...' part should be the actual block0 hash of the network_

``'asdf1234 ...'부분은 네트워크의 실제 block0 해시 여야합니다.``
