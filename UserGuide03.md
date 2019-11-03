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

- `level`: 로깅 레벨. 기본값은 "info". 선택 가능 "off", "critical", "error", "warn", "info", "debug", "trace"

- `format`: log output format - `plain` or `json`

- `output`: log output - `stdout`, `stderr`, `syslog` (Unix only),
  or `journald` (Linux 만 사용가능하며, 컴파일 할때 활성화해야 사용 가능함).


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
- `pkcs12`: (optional) certificate file 
- `cors`: (optional) CORS 구성, 설정하지 않으면 CORS 는 비활성화됨
  - `allowed_origins`: (optional) allowed origins, 설정하지 않으면 origin 으로 설정됨
  - `max_age_secs`: (optional) 최대 CORS 캐싱 시간(초), 설정하지 않으면, 캐싱은 비활성화됨

## P2P configuration

- `trusted_peers`: (optional) p2p 토폴로지(and bootstrap our local blockchain)를 연관된 `public_id` 로 부트스트랩 하기 위해 연결할 노드의 [multiaddr][multiaddr] 목록
    

- `public_address`: [multiaddr][multiaddr] 는 수신하고 연결을 허락할 주소.
    이 노드가 다른 피어에게 블록체인 배포를 원할 경우, 네트워크의 다른 피어에게 배포될 이 노드의 주소

- `private_id`: (optional) `Ed25519` 유형의 암호 비밀키. 키를 생성하는 자세한 내용은 [`jcli key`] 참조. 
    지정하지 않으면 임의의 키가 생성됨. 연관된 공개키는 네트워크에서 노드의 고유식별자가 됨

- `listen_address`: (optional) [multiaddr][multiaddr] 는 노드가 p2p 연결 수신을 위한 주소 지정.
    지정하지 않으면 public_address에 지정된 값으로 p2p 연결을 수신함

- `topics_of_interest`: 노드가 관심 있어 하는 다양한 항목:
    - `messages`: 이 노드가 트랜잭션에 관심이 있는 다른 피어에게 알림.
    일반적인 설정 passive node: `low`. stakepool: `high`
    - `blocks`: 이 노드가 새 블록에 관심이 있는 다른 피어에게 알림.
    일반적인 설정 passive node: `normal`. stakepool: `high`

- `max_connections`: 이 노드가 유지해야 하는 최대 p2p 연결수. 지정하지 않는 경우, 기본 값은 internal limit 이 적용됨

[multiaddr]: https://github.com/multiformats/multiaddr

[`jcli key`]: ../jcli/key.md

When running an active node (BFT leader or stake pool) it is interesting to be
able to make choices on how to manage the pending transactions: how long to keep
them, how to prioritize them etc.

``활성 노드 (BFT 리더 또는 스테이크 풀)를 실행할 때 보류중인 트랜잭션을 관리하는 방법, 유지 기간, 우선 순위 지정 방법 등을 선택할 수 있는 것이 흥미롭습니다.``

The `mempool` field in your node config file is not mandatory, by default it is set
as follow:

``노드 구성 파일의 mempool 필드는 필수는 아니며 기본적으로 다음과 같이 설정됩니다.``

```yaml
mempool:
    fragment_ttl: 30m
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `fragment_ttl` 은 노드가 풀에서 프래그먼트(a _transaction_)를 얼마나 보관해야 하는지를 기술합니다
* `log_ttl` 은 노드가 풀에서 보류/수락/거부 된 프래그먼트 로그를 보관하는 기간을 설명합니다
* `garbage_collection_interval`  노드가 시간 초과된 항목(프래그먼트 또는 로그)을 제거 할 때의 간격을 설명합니다


The `leadership` field in your node config file is not mandatory, by default it is set
as follow:

``노드 구성 파일의 리더십 필드는 필수는 아니며 기본적으로 다음과 같이 설정됩니다.``

```yaml
leadership:
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `log_ttl` 은 노드가 리더 이벤트 로그를 보관하는 기간을 기술합니다.
* `garbage_collection_interval` 은 2개의 가비지 수집 실행 간격(예: 노드가 시간 초과된 item log 를 제거하는 경우)
