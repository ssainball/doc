# Configuration

This chapter covers the node documentation, necessary to have a working system. It covers
the network, logging and storage parameters.

``이 장에서는 작업 시스템을 갖추는 데 필요한 노드 설명서를 다룹니다. 네트워크, 로깅, 스토리지 매개 변수를 다룹니다.``

The node configuration uses the [YAML](https://en.wikipedia.org/wiki/YAML) format.

``노드 구성은 YAML 형식을 사용합니다.``

This is an example of a configuration file:

``다음은 구성 파일의 예입니다.``

```YAML
storage: "/tmp/storage"
log:
  level: debug
  format: json
p2p:
  trusted_peers:
    - address: "/ip4/104.24.28.11/tcp/8299"
      id: ed25519_pk1tyz2drx5kvsp49fk0gunkl8ktlhjc976zj5erqwavnpfmrexxxnscxln75
    - address: "/ip4/104.24.29.11/tcp/8299"
      id: ed25519_pk13j4eata8e567xwdqp6wjeu8wa7dsut3kj0u3tgulrsmyvveq9qxqeqr3kc
  public_address: "/ip4/127.0.0.1/tcp/8080"
  private_id: ed25519_sk1n649x7zrmt38q6dtqkvj769wru4vpkfnrppw6d83qd7jh3uhux7qwhg8q3
  topics_of_interest:
    messages: low
    blocks: normal
```
The following options are available in the log section:

``로그 섹션에서 다음 옵션을 사용할 수 있습니다.``

- `level`: log messages minimum severity. If not configured anywhere, defaults to "info".
  Possible values: "off", "critical", "error", "warn", "info", "debug", "trace".
- `format`: log output format - `plain` or `json`.
- `output`: log output - `stdout`, `stderr`, `syslog` (Unix only),
  or `journald` (Linux with systemd only, must be enabled during compilation).


There's 2 differents network interfaces which are covered by their respective section:

``각 섹션에서 다루는 두 가지 다른 네트워크 인터페이스가 있습니다.``

```yaml
rest:
   ...
p2p:
   ...
```

## REST interface configuration

- `listen`: listen address
- `pkcs12`: certificate file (optional)
- `cors`: (optional) CORS configuration, if not provided, CORS is disabled
  - `allowed_origins`: (optional) allowed origins, if none provided, echos request origin
  - `max_age_secs`: (optional) maximum CORS caching time in seconds, if none provided, caching is disabled

## P2P configuration

- `trusted_peers`: (optional) the list of nodes' [multiaddr][multiaddr] to connect to in order to
    bootstrap the p2p topology (and bootstrap our local blockchain) with the associated `public_id`.
- `public_address`: [multiaddr][multiaddr] the address to listen from and accept connection
    from. This is the public address that will be distributed to other peers
    of the network that may find interest into participating to the blockchain
    dissemination with the node;
- `private_id`: (optional) a cryptographic secret key of type `Ed25519`. See [`jcli key`] for more info
  on how to generate a key. If not set, a random key will be generated.
  The associated public key is the node's unique identifier on the network.
- `listen_address`: (optional) [multiaddr][multiaddr] specifies the address the node
    will listen to to receive p2p connection. Can be left empty and the node will listen
    to whatever value was given to `public_address`.
- `topics_of_interest`: the different topics we are interested to hear about:
    - `messages`: notify other peers this node is interested about Transactions
    typical setting for a non mining node: `"low"`. For a stakepool: `"high"`;
    - `blocks`: notify other peers this node is interested about new Blocs.
    typical settings for a non mining node: `"normal"`. For a stakepool: `"high"`.
- `max_connections`: the maximum number of P2P connections this node should
    maintain. If not specified, an internal limit is used by default.

[multiaddr]: https://github.com/multiformats/multiaddr

[`jcli key`]: ../jcli/key.md

When running an active node (BFT leader or stake pool) it is interesting to be
able to make choices on how to manage the pending transactions: how long to keep
them, how to prioritize them etc.

``활성 노드 (BFT 리더 또는 스테이크 풀)를 실행할 때 보류중인 트랜잭션을 관리하는 방법, 유지 기간, 우선 순위 지정 방법 등을 선택하는 것이 흥미 롭습니다.``

The `mempool` field in your node config file is not mandatory, by default it is set
as follow:

``노드 구성 파일의 mempool 필드는 필수는 아니며 기본적으로 다음과 같이 설정됩니다.``

```yaml
mempool:
    fragment_ttl: 30m
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `fragment_ttl` describes for how long the node shall keep a fragment (a _transaction_)
  pending in the pool before being discarded;
* `log_ttl` describes for how long the node will keep logs of pending/accepted/rejected
  fragments in the pool; This is link to the data you receives from the REST fragment
  logs end point;
* `garbage_collection_interval` describes the interval between 2 garbage collection
  runs: i.e. when the node removes item (fragments or logs) that have timed out. 

``* fragment_ttl은 노드가 폐기되기 전에 프래그먼트(트랜잭션)가 풀에 보류되는 시간을 설명합니다. ``

``* log_ttl은 노드가 풀에서 보류/수락/거부 된 조각에 대한 로그를 보관하는 기간을 설명합니다. REST 프레이그먼트 로그에서 수신 한 데이터에 대한 링크입니다.``

``* garbage_collection_interval은 2 개의 가비지 수집 실행 간격, 즉 노드가 시간 초과 된 항목 (조각 또는 로그)을 제거 할 때의 간격을 설명합니다.``


The `leadership` field in your node config file is not mandatory, by default it is set
as follow:

``노드 구성 파일의 리더십 필드는 필수는 아니며 기본적으로 다음과 같이 설정됩니다.``

```yaml
leadership:
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `log_ttl` describes for how long the node will keep logs of leader events.
  This is link to the data you receives from the REST leadership logs end point;
* `garbage_collection_interval` describes the interval between 2 garbage collection
  runs: i.e. when the node removes item logs that have timed out
  
 ``* log_ttl은 노드가 리더 이벤트 로그를 유지하는 기간을 설명합니다. REST 리더십 로그 엔드 포인트에서 수신 한 데이터에 대한 링크입니다.``
  
 ``* garbage_collection_interval은 두 가비지 수집 실행 간격을 설명합니다. 즉, 노드가 시간 초과 된 항목 로그를 제거 할 때``
