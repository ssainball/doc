# Jormungandr User Guide

Welcome to the Jormungandr User Guide.

``Jormungandr 사용자 가이드에 오신 것을 환영합니다.``

Jormungandr is a node implementation, written in rust, with the
initial aim to support the Ouroboros type of consensus protocol.

``Jormungandr는 Ouroboros 유형의 합의 프로토콜을 지원하기 위한 초기 목표를 가진 'RUST' 언어로 작성된 노드 구현입니다.``


A node is a participant of a blockchain network, continuously making,
sending, receiving, and validating blocks. Each node is responsible
to make sure that all the rules of the protocol are followed.

``노드는 블록체인 네트워크의 참여자로서 블록을 지속적으로 만들고, 보내고, 받고, 검증합니다. 각 노드는 프로토콜의 모든 규칙을 준수하는지 확인할 책임이 있습니다.``

## Mythology

Jörmungandr refers to the _Midgard Serpent_ in Norse mythology. It is a hint to
_Ouroboros_, the Ancient Egyptian serpent, who eat its own tail, as well as the
[IOHK paper](https://eprint.iacr.org/2016/889.pdf) on proof of stake.

``요르문간드는 북유럽 신화에서 미드가르드 뱀을 가리 킵니다. 그것은 자신의 꼬리를 먹는 고대 이집트 뱀 우로보로스와 IOHK의 지분 증명 논문에게도 실마리를 제공합니다.``


# General Concepts

This chapter covers the general concepts of the blockchain, and their application
in the node, and is followed by the node organisation and the user interaction with it.

``이 장에서는 블록체인의 일반적인 개념, 노드에서의 적용, 노드 구성, 사용자와의 상호 작용을 설명합니다.``

# Blockchain concepts

## Time

Slots represent the basic unit of time in the blockchain, and at each slot
a block could be present.

``슬롯은 블록체인의 기본 시간 단위를 나타내며 각 슬롯에는 블록이 존재할 수 있습니다.``

Consecutive slots are grouped into epochs, which have updatable size defined
by the protocol.

``연속 슬롯은 프로토콜에 정의에 의해 업데이트 가능한 크기를 갖는 에포크로 그룹화 됩니다.``


## Fragments

Fragments are part of the blockchain data that represent all the possible
events related to the blockchain health (e.g. update to the protocol), but
also and mainly the general recording of information like transactions and
certificates.

``프래그먼트는 블록체인 상태(예 : 프로토콜 업데이트)와 관련된 모든 가능한 이벤트를 나타내는 블록체인 데이타의 일부분이지만, 거래 나 인증서와 같은 정보의 일반적인 기록도 많이 나타냅니다.``

## Blocks

Blocks represent the spine of the blockchain, safely and securely linking
blocks in a chain, whilst grouping valid fragments together.

``블록은 블록체인의 척추이며, 유효한 프래그먼트를 함께 그룹화하면서 블록을 안전하고 보안적으로 연결합니다.``

Blocks are composed of 2 parts:

``블록은 두 부분으로 구성됩니다.``

* The header
* The content

The header link the content with the blocks securely together, while the
content is effectively a sequence of fragments.

``헤더는 내용물을 블록과 안전하게 함계 연결하는 반면 내용물은 사실상 프래그먼트의 입니다.``

## Blockchain

The blockchain is the general set of rules and the blocks that are periodically created.
Some of the rules and settings, can be changed dynamically in the system by updates,
while some other are hardcoded in the genesis block (first block of the blockchain).

``블록체인은 주기적으로 생성되는 규칙과 블록의 집합입니다. 일부 규칙 및 설정은 업데이트를 통해 시스템에서 동적으로 변경 될 수 있습니다. 또 다른 규칙 및 설정은 생성 제네시스 블록에 하드 코딩됩니다.``

```
    +-------+      +-------+
    |Genesis+<-----+Block 1+<--- ....
    |Header |      |Header |
    +---+---+      +---+---+
        |              |
    +---v---+      +---v---+
    |Genesis|      |Block 1|
    |Content|      |Content|
    +-------+      +-------+
```

## Consensus

The node currently support the following consensus protocol:

``노드는 현재 다음 합의 프로토콜을 지원합니다.``

* Ouroboros BFT (OBFT)
* Ouroboros Genesis-Praos

Ouroboros BFT is a simple Byzantine Fault Tolerant (BFT) protocol where the
block makers is a known list of leaders that successively create a block and
broadcast it on the network.

``Ouroboros BFT는 블록 메이커가 블록을 생성하고 네트워크에 브로드캐스트 하는 leader 목록 프로토콜 입니다. ``

Ouroboros Genesis Praos is a proof of stake (PoS) protocol where the block
maker is made of a lottery where each stake pool has a chance proportional to
their stake to be elected to create a block. Each lottery draw is private to
each stake pool, so that the overall network doesn't know in advance who can
or cannot create blocks.

``Ouroboros Genesis Praos 는 블록 메이커가 스테이크에 비례하여 블록 생성자로 뽑힐 수 있는 복권 프로토콜 입니다. 각 복권 추첨은 각 이해관계자(스테이크 풀) 풀에 대해 비공개로 진행되므로, 전체 네트워크가 누가 블록을 만들 수 있는지, 만들지 못하는지 여부를 미리 알 수 없습니다.``

In Genesis-Praos slot time duration is constant, however the frequency of 
creating blocks is not stable, since the creation of blocks is a probability 
that is linked to the stake and consensus_genesis_praos_active_slot_coeff.

``Genesis-Praos에서 슬롯 지속 시간은 일정하지만 블록 생성 빈도는 일정하지 않습니다. 블록의 생성은 스테이크 및 consensus_genesis_praos_active_slot_coeff 와 연결된 확률이기 때문입니다.``

**Note**: In Genesis-Praos, if there is no stake in the system, no blocks will be 
created anymore starting with the next epoch.

``참고 : Genesis-Praos에서 시스템에 스테이크가 없으면 다음 에포크부터 더 이상 블록이 생성되지 않습니다.``

## Leadership

The leadership represent in abstract term, who are the overall leaders of the
system and allow each individual node to check that specific blocks are
lawfully created in the system.

``리더십은 추상적인 용어로 표현되며, 그들은 시스템의 전체리더로서, 각 개별 노드가 특정 블록이 시스템에서 합법적으로 생성되었는지 확인할 수 있도록 합니다.``

The leadership is re-evaluated at each new epoch and is constant for the
duration of an epoch.

``리더십은 각각의 새로운 에포크에서 재평가되며, 에포크 동안 지속됩니다.``

## Leader

Leader are an abstraction related to the specific actor that have the ability
to create block; In OBFT mode, the leader just the owner of a cryptographic
key, whereas in Genesis-Praos mode, the leader is a stake pool.

``리더는 블록을 생성할 수 있는 어떤 생성자와 관련된 추상화입니다. OBFT 모드에서는 리더가 암호 키의 소유자 일뿐 아니라 Genesis-Praos 모드에서는 리더가 스테이크 풀입니다.``

## Transaction

Transaction forms the cornerstone of the blockchain, and is one type of fragment
and also the most frequent one.

``거래는 블록체인의 초석을 이루며, 한 가지 유형의 프래그먼트이자 가장 빈번한 것입니다.``

Transaction is composed of inputs and outputs; On one side, the inputs represent
coins being spent, and on the other side the outputs represent coins being received.

``트랜잭션은 입력과 출력으로 구성됩니다. 한쪽은 입력된 동전을 나타내고 다른 쪽은 수신되는 동전을 나타냅니다.``


```
    Inputs         Alice (80$)        Bob (20$)
                        \             /
                         \           /
                          -----------
                                100$
                             --------- 
                            /         \
    Outputs            Charlie (50$)  Dan (50$)
```

Transaction have fees that are defined by the blockchain settings and the following invariant hold:

``거래에는 블록체인 설정 및 다음과 같이 일정한 수수료가 정의되어 있습니다.``

\\[ \sum Inputs = \sum Outputs + fees \\]

Transaction need to be authorized by each of the inputs in the transaction by their respective witness.
In the most basic case, a witness is a cryptographic signature, but depending on the type of input can
the type of witness vary.

``거래는 각 증인에 의해 거래마다 입력 승인이 필요합니다. 가장 기본적인 경우 감시는 암호화 서명이지만 입력 유형에 따라 감시 유형이 다를 수 있습니다.``

## Accounting

The blockchain has two methods of accounting which are interoperable:

``블록체인에는 상호 운용 가능한 두 가지 회계 방법이 있습니다.``

* Unspent Transaction Output (UTXO)
* Accounts

UTXO behaves like cash/notes, and work like fixed denomination ticket that are
cumulated. This is the accounting model found in Bitcoin. A UTXO is uniquely
reference by its transaction ID and its index.

``UTXO는 현금 장부와 같은 누적된 고정 금액의 티켓처럼 작동합니다. 이것은 비트코인에서 발견된 회계 모델입니다. UTXO는 트랜잭션 ID와 색인으로 고유하게 참조됩니다.``

Accounts behaves like a bank account, and are simpler to use since exact amount
can be used. This is the accounting model found in Ethereum. An account is
uniquely identified by its public key.

``계정은 은행 계좌처럼 작동하며 정확한 금액을 사용할 수 있으므로 사용하기가 더 쉽습니다. 이것은 이더리움에서 발견된 회계 모델입니다. 계정은 공개 키로 고유하게 식별됩니다.``

Each inputs could refer arbitrarily to an account or a UTXO, and similarly
each outputs could refer to an account or represent a new UTXO.

``각 입력은 임의로 계정 또는 UTXO를 참조할 수 있습니다. 마찬가지로 각 출력은 계정을 참조하거나 새로운 UTXO를 나타낼 수 있습니다.``


# Stake

In a proof of stake, participants are issued a stake equivalent to the amount
of coins they own. The stake is then used to allow participation in the protocol,
simply explained as:

``스테이크 증거로 참가자는 자신이 소유한 코인의 양에 해당하는 스테이크를 발행합니다. 그런 다음 스테이크는 프로토콜에 참여하는 데 사용되며 다음과 같이 간단히 설명됩니다.``

> The more stake one has, the more likely one will participate in the good health of the network.

> 스테이크가 많을수록 네트워크의 건강 상태에 참여할 가능성이 높습니다.

When using the BFT consensus, the stake doesn't influence how the system
runs, but stake can still be manipulated for a later transition of the chain
to another consensus mode.

``BFT 합의를 사용할 때 스테이크는 시스템 운영 방식에 영향을 미치지 않지만 체인을 나중에 다른 합의 모드로 전환하기 위해 스테이크를 조작할 수 있습니다.``

## Stake in the Account Model

Account are represented by 1 type of address and are just composed of a public key.
The account accumulate moneys and its stake power is directly represented by the amount it contains

``계정은 1가지 유형의 주소로 표시되며 공개 키로만 구성됩니다. 계정은 돈을 축적하고 계정 지분은 해당 금액으로 직접 표시됩니다.``

For example:

```

    A - Account with 30$ => Account A has stake of 30
    B - Account with 0$ => Account B has no stake

```

The account might have a bigger stake than what it actually contains, since it could
also have associated UTXOs, and this case is covered in the next section.

``계정에 UTXO가 연결될 수 있으므로 계정에 실제로 포함된 것보다 더 큰 지분이 있을 수 있으며 이 사례는 다음 섹션에서 다룹니다.``


## Stake in the UTXO Model

UTXO are represented by two kind of addresses:

``UTXO 는 두 가지 종류의 주소로 표시됩니다.``

* single address: 이러한 유형의 주소에는 스테이크가 연결되어 있지 않습니다
* group address: 이러한 유형의 주소에는 UTXO 지분 가치가 있는 계정이 연결되어 있습니다.


For example with the following utxos:

```
    UTXO1 60$ (single address) => has stake of 0

    UTXO2 50$ (group address A) \
                                 ->- A - Account with 10$ => Account A has stake of 100
    UTXO3 40$ (group address A) /

    UTXO4 20$ (group address B) -->- B - Account with 5$ => Account B has stake of 25
```

## Stake pool

Stake pool are the trusted block creators in the genesis-praos system. A pool
is declared on the network explicitely by its owners and contains, metadata
and cryptographic material.

``스테이크 풀은 genesis-praos 시스템에서 신뢰할 수 있는 블록 제작자입니다. 풀은 그것의 소유자에 의해 명시적으로 네트워크에 선언하고, 메타 데이터와 암호화된 자료를 포함합니다.``

Stake pool has no stake power on their own, but participants in the network
delegate their stake to a pool for running the operation.

``스테이크 풀은 자체적으로 스테이크 권한이 없지만 네트워크의 참가자는 운영을 위해 풀에 스테이크를 위임합니다.``


## Stake Delegation

Stake can and need to be delegated to stake pool in the system. They can
change over time with a publication of a new delegation certificate.

``스테이크는 스테이크 풀에 위임 될 수 있습니다. 새 위임 인증서를 게시하면 시간이 지남에 따라 변경 될 수 있습니다.``

Delegation certificate are a simple declaration statement in the form of:

``위임 인증서는 다음과 같은 형식의 간단한 선언문입니다.``

```
    Account 'A' delegate to Stake Pool 'Z'
```

Effectively it assign the stake in the account and its associated UTXO stake
to the pool it delegates to until another delegation certificate is made.

``효과적으로 다른 위임 인증서가 작성 될 때까지 계정의 지분 및 관련 UTXO 지분을 위임하는 풀에 할당합니다.``


# Node organisation


## Secure Enclave

The secure enclave is the component containing the secret cryptographic
material, and offering safe and secret high level interfaces to the rest of
the node.

`` secure enclave 는 비밀 암호화 자료를 포함하고 나머지 노드에 안전하고 비밀이 높은 수준의 인터페이스를 제공하는 구성 요소입니다.``

## Network

The node's network is 3 components:

``노드의 네트워크 3 가지 구성 요소입니다.``

* Intercommunication API (GRPC)
* Public client API (REST)
* Control client API (REST)

More detailed information [here](./network.md)

### Intercommunication API (GRPC)

This interface is a binary, efficient interface using the protobuf format and
GRPC standard. The protobuf files of types and interfaces are available in
the source code.

``이 인터페이스는 protobuf 형식과 GRPC 표준을 사용하는 이진 효율적인 인터페이스입니다. 유형 및 인터페이스의 프로토 타입 파일은 소스 코드에서 사용 가능합니다.``

The interface is responsible to communicate with other node in the network:

``인터페이스는 네트워크의 다른 노드와 통신해야합니다.``

* block sending and receiving
* fragments (transaction, certificates) broadcast
* peer2peer gossip

### Public API REST

This interface is for simple queries for clients like:

``이 인터페이스는 다음과 같은 클라이언트에 대한 간단한 쿼리를 위한 것입니다.``

* Wallet Client & Middleware
* Analytics & Debugging tools
* Explorer

it's recommended for this interface to not be opened to the public.

``이 인터페이스를 공개하지 않는 것이 좋습니다.``

TODO: Add a high level overview of what it does

``앞으로 할 일 : 기능에 대한 개괄적인 개요 추가``

### Control API REST

This interface is not finished, but is a restricted interface with ACL,
to be able to do maintenance tasks on the process:

``이 인터페이스는 완료되지 않았지만 프로세스에서 유지 보수 작업을 수행 할 수 있도록 ACL이 있는 제한된 인터페이스입니다.``

* Shutdown
* Load/Retire cryptographic material

TODO: Detail the ACL/Security measure

``앞으로 할 일 : ACL/보안조치 세부 사항``

Jörmungandr network capabilities are split into:

``Jörmungandr 네트워크 기능은 다음과 같이 나뉩니다.``

1. the REST API, 정보 조회 또는 노드 제어에 사용됨;
2. 블록체인 프로토콜 교환 및 참여를 위한 gRPC API;


Here we will only talk of the later, the REST API is described in another
chapter already: [go to REST documentation](../quickstart/03_rest_api.md)

``여기서는 나중에 이야기 할 것입니다. REST API는 이미 다른 장에서 설명 합니다. REST 문서로 이동``


# The protocol

The protocol is based on commonly used in the industry tools: HTTP/2 and RPC.
More precisely, Jörmungandr utilises [`gRPC`](https://www.grpc.io).

``이 프로토콜은 산업 도구에서 일반적으로 사용되는 HTTP/2 및 RPC를 기반으로 합니다. 좀 더 정확하게 말씀드리면 Jörmungandr는 gRPC를 활용 합니다.``

This choice has been made for it is already widely supported across the world,
it is utilising HTTP/2 which makes it easier for Proxy and Firewall to recognise
the protocol and allow it.

``이 선택은 이미 전 세계에서 널리 지원되고 있기 때문에 프록시와 방화벽이 프로토콜을 인식하고 허용하는 것을 보다 쉽게하는 HTTP/2를 사용하고 있습니다.``

## Type of queries

The protocol allows to send multiple types of messages between nodes:

* sync block to remote peer's _Last Block_ (`tip`).
* propose new fragments (new transactions, certificates, ...):
  this is for the fragment propagation.
* propose new blocks: for the block propagation.

``이 프로토콜을 사용하면 노드간에 여러 유형의 메시지를 보낼 수 있습니다.``

* remote peer 의 _Last Block_ (tip) 동기화
* 새로운 프래그먼트(새 트랜잭션, 인증서) 제안. 이것은 프래그먼트 전파를 위한 것입니다.
* 새로운 블록 제안 (블록 전파를 위해)

There are other commands to optimise the communication and synchronisation
between nodes.

``노드 간 통신 및 동기화를 최적화하는 다른 명령이 있습니다.``

Another type of messages is the `Gossip` message. It allows Nodes to exchange
information (gossips) about other nodes on the network, allowing the peer
discovery.

``또 다른 유형의 메시지는 Gossip 메시지입니다. 이를 통해 노드는 네트워크의 다른 노드에 대한 정보(가십)를 교환하여 피어 발견을 허용합니다.``

## Peer discovery

Peer discovery is done via [`Poldercast`](https://hal.inria.fr/hal-01555561/document)'s Peer to Peer (P2P) topology
construction. The idea is to allow the node to participate actively into
building the decentralized topology of the p2p network.

``피어 검색은 Poldercast의 P2P 토폴로지(체계적 분류) 구성을 통해 수행됩니다. 아이디어는 노드가 p2p 네트워크의 분산 토폴로지(체계적 분류)를 구축하는 데 적극적으로 참여할 수 있도록 하는 것입니다.``

This is done through gossiping. This is the process of sharing with others
topology information: who is on the network, how to reach them and what are
they interested about.

``이것은 가십을 통해 이루어집니다. 이것은 다른 토폴로지 정보를(네트워크에있는 사람, 연락하는 방법 및 관심있는 대상) 공유하는 프로세스입니다.``

In the poldercast paper there are 3 different modules implementing 3 different
strategies to select nodes to gossip to and to select the gossiping data:

1. Cyclon: this module is responsible to add a bit of randomness in the gossiping
   strategy. It also prevent nodes to be left behind, favouring contacting Nodes
   we have the least used;
2. Vicinity: this module helps with building an interest-induced links between
   the nodes of the topology. Making sure that nodes that have common interests
   are often in touch.
3. Rings: this module create an oriented list of nodes. It is an arbitrary way to
   link the nodes in the network. For each topics, the node will select a set of
   close nodes.

`` poldercast 논문에는 가십 노드를 선택하고 gossiping 데이터를 선택하는 3가지 전략을 구현하는 3가지 모듈이 있습니다.``

1. Cyclon: ``이 모듈은 gossiping 전략에 약간의 무작위성을 추가합니다. 또한 노드가 남겨지는 것을 방지하여 가장 적게 사용 된 노드와의 접촉을 선호합니다.``

2. Vicinity: ``이 모듈은 토폴로지(체계적 분류) 노드 사이에 관심 유도 링크를 작성하는 데 도움이 됩니다. 공통의 관심사를 가진 노드가 종종 연락해야합니다.``

3. Rings: ``이 모듈은 지향적인 노드 목록을 만듭니다. 네트워크의 노드를 연결하는 임의의 방법입니다. 각 주제에 대해 노드는 closed node set를 선택합니다.``




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

1. the hash of the **genesis block** of the blockchain, this will be the source
   of truth of the blockchain. It is 64 hexadecimal characters.
2. the **trusted peers** identifiers and access points.


``노드를 시작하려면 먼저 연결해야하는 블록체인 정보를 수집해야 합니다.``

1. ``제네시스 블록 해시, 이것은 블록체인 신뢰의 근원입니다. 64개의 16진수 입니다.``
2. ``트러스트 피어 식별자 그리고 접근 포인트.``

These information are essentials to start your node in a secure way.

``이 정보는 노드를 안전하게 시작하는 데 필수적입니다.``

The **genesis block** is the first block of the blockchain. It contains the
static parameters of the blockchain as well as the initial funds. Your node
will utilise the **Hash** to retrieve it from the other peers. It will also
allows the Node to verify the integrity of the downloaded **genesis block**.

``제네시스 블록은 블록체인의 첫 번째 블록입니다. 초기 자금규모뿐만 아니라 블록체인의 정적 매개 변수가 포함되어 있습니다. 노드는 해시를 사용하여 다른 피어에서 검색합니다. 또한 노드가 다운로드된 제네시스 블록의 무결성을 확인할 수 있습니다.``

The **trusted peers** are the nodes in the public network that your Node will
trust in order to initialise the Peer To Peer network.

`` 트러스트 피어는 당신의 노드를 초기화하기 위해 퍼블릭 네트워크에서 믿을 수 있는 노드입니다.``

# The node configuration

Your node configuration file may look like the following:

``노드 구성 파일은 다음과 같습니다.``

**Note**

This config shouldn't work as it is, the ip address and port for the trusted peer should be those of an already running node.
Also, the public_address ('u.x.v.t') should be a valid address (you can use an internal one, eg: 127.0.0.1).
Furthermore, you need to have permission to write in the path specified by the storage config.

``이 구성을 그대로 복사해서 사용하면 작동하지 않습니다. 트러스트된 피어의 IP 주소 및 포트는 이미 실행중인 노드의 IP 주소 및 포트 여야합니다. 또한 public_address('u.x.v.t')는 유효한 주소 여야합니다(예 : 127.0.0.1). 또한 스토리지 구성에 지정된 경로에 쓸 수 있는 권한이 있어야 합니다.``

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

- `storage`: (optional) Path to the storage. If omitted, the
  blockchain is stored in memory only.
- `log`: (optional) Logging configuration:
    - `level`: log messages minimum severity. If not configured anywhere, defaults to "info".
        Possible values: "off", "critical", "error", "warn", "info", "debug", "trace".
    - `format`: Log output format, `plain` or `json`.
    - `output`: Log output destination. Possible values are:
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
      - `allowed_origins`: (optional) allowed origins, if none provided, echos request origin
      - `max_age_secs`: (optional) maximum CORS caching time in seconds, if none provided, caching is disabled
- `p2p`: P2P network settings
    - `trusted_peers`: (optional) the list of nodes's [multiaddr][multiaddr] with their associated `public_id`
      to connect to in order to bootstrap the P2P topology (and bootstrap our local blockchain);
    - `private_id`: the node's private key ([`Ed25519`]) that will be used to identify this node to the network
    - `public_address`: [multiaddr][multiaddr] string specifying address of the
      P2P service. This is the public address that will be distributed to other
      peers of the network that may find interest in participating to the
      blockchain dissemination with the node.
    - `listen_address`: (optional) [multiaddr][multiaddr] specifies the address the node
      will listen to to receive p2p connection. Can be left empty and the node will listen
      to whatever value was given to `public_address`.
    - `topics_of_interest`: The dissemination topics this node is interested to hear about:
      - `messages`: Transactions and other ledger entries.
        Typical setting for a non-mining node: `low`. For a stakepool: `high`;
      - `blocks`: Notifications about new blocks.
        Typical setting for a non-mining node: `normal`. For a stakepool: `high`.
    - `max_connections`: The maximum number of simultaneous P2P connections
      this node should maintain.
- `explorer`: (optional) Explorer settings
    - `enabled`: True or false

[multiaddr]: https://github.com/multiformats/multiaddr


필드 설명 :

- `storage`: (옵션) 저장소 경로입니다. 생략하면 블록체인은 메모리에만 저장됩니다.
- `log`: (옵션) 로깅 구성:
    - `level`: 로그 메세지 최소 심각도. 설정하지 않으면 기본 값은 "info" 입니다.
        가능 값: "off", "critical", "error", "warn", "info", "debug", "trace".
    - `format`: 로그 아웃 포맷, `plain` or `json`.
    - `output`: 로그 출력 대상. 가능한 값은 다음과 같습니다.:
      - `stdout`: 표준 출력
      - `stderr`: 표준 에러
      - `syslog`: syslog (Unix 만 가능)
      - `journald`: journald service (Linux 만 가능, jormungandr 가 `systemd` 기능으로 빌드 된 경우)
      - `gelf`: GELF (Graylog) network 로깅 프로토콜의 구성 필드 (jormungandr가 `gelf` 기능으로 빌드 된 경우):
        - `backend`: _hostname_:_port_  GELF server 구성
        - `log_id`: 메시지의`host` 필드에 대한 로그 소스의 식별자.
- `rest`: (옵션) REST endpoint 구성.
    - `listen`: _address_:_port_ 요청을 듣기 위한 구성
    - `pkcs12`: (옵션) Certificate file
    - `cors`: (옵션) CORS configuration, if not provided, CORS is disabled
      - `allowed_origins`: (옵션) 허용 된 원점 (제공된 경우)
      - `max_age_secs`: (옵션) 최대 CORS 캐싱 시간 (초) (제공된 것이 없으면 캐싱이 비활성화 됨)
- `p2p`: P2P network settings
    - `trusted_peers`: (옵션) P2P 토폴로지와 로컬 블록체인을 부트스트랩하기 위해 연결할 multiaddr(노드 public_id) 목록입니다.;
    - `private_id`: 네트워크에서 이 노드를 식별하는 데 사용될 노드의 개인 키 (Ed25519)
    - `public_address`: P2P 서비스의 주소 지정. 이것은 노드의 블록체인 보급에 관심이 있는 네트워크의 다른 피어에게 배포 될 공개 주소입니다.
    - `listen_address`: (옵션) multiaddr은 p2p 연결을 수신하기 위해 노드가 수신 할 주소를 지정합니다. 비워 둘 수 있으며 노드는 public_address에 지정된 값을 수신합니다.
    - `topics_of_interest`: 이 노드의 메세지 전파 항목은 다음과 같습니다. :
      - `messages`: 거래 및 기타 원장 항목.
        non-mining node: `low`.  stakepool: `high`; 
      - `blocks`: 새로운 블록에 대한 알림.
        non-mining node: `normal`. stakepool: `high`.
    - `max_connections`: 이 노드가 유지해야하는 최대 동시 P2P 연결 수입니다.
- `explorer`: (옵션) Explorer settings
    - `enabled`: True or false

[multiaddr]: https://github.com/multiformats/multiaddr

# Starting the node

```
jormungandr --config config.yaml --genesis-block-hash 'abcdef987654321....'
```

The 'abcdef987654321....' part refers to the hash of the genesis, that should be given to you from one of the peers in the network you are connecting to.

``'abcdef987654321 ....' 부분은 연결중인 네트워크의 피어 중 하나에서 제공해야 하는 제네시스의 해시를 나타냅니다.``

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

`` 접속 지점(end points) 및 결과는 향후 변경 될 수 있습니다.``

To see the whole Node API documentation,
[click here](https://editor.swagger.io/?url=https://raw.githubusercontent.com/input-output-hk/jormungandr/master/doc/openapi.yaml)

``전체 노드 API 설명서를 보려면 여기를 클릭하십시오``

# Explorer mode

The node can be configured to work as a explorer. This consumes more resources, but makes it possible to query data otherwise not available.

``탐색기로 작동하도록 노드를 구성할 수 있습니다. 이로 인해 더 많은 리소스가 소비되지만 접근 불가했던 데이터를 쿼리할 수 있습니다.``

## Configuration

There is two ways of enabling the explorer api. It can either be done by passing the `--enable-explorer` flag on the start arguemnts or by the config file: 

``탐색기 API를 활성화하는 방법에는 두 가지가 있습니다. 시작 인수에 '--enable-explorer' 플래그를 전달하거나 구성 파일을 사용하여 수행할 수 있습니다.``

``` yaml
explorer:
    enabled: true
```

#### CORS

For configuring CORS the explorer API, this needs to be done on the REST section of the config, as documented [here](../configuration/network.md).

``CORS 탐색기 API를 구성하려면 여기에 설명된 대로 환경설정의 REST 섹션에서 수행해야 합니다.``

## API

A graphql interface can be used to query the explorer data, when enabled, two endpoints are available in the [REST interface](03_rest_api.md): `/explorer/graphql` and `/explorer/graphiql` .

``graphql 인터페이스를 사용하여 탐색기 데이터를 조회 할 수 있습니다. 사용 가능한 경우 REST 인터페이스에서 '/explorer/graphql' 및 '/explorer /graphiql' 의 두 엔드 포인트를 사용할 수 있습니다.``

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

1. the hash of the **genesis block** of the blockchain, this will be the source
   of truth of the blockchain. It is 64 hexadecimal characters.
2. the **trusted peers** identifiers and access points.


``패시브 노드 사례처럼 기존 네트워크에 연결하려면 두 가지가 필요합니다.``

1. ``제네시스 블록 해시, 이것은 블록체인 신뢰의 근원입니다. 64개의 16진수 입니다.``
2. ``트러스트 피어 식별자 그리고 접근 포인트.``


The node configuration could be the same as that for [running a passive node](./01_passive_node.md). 

``노드 구성은 수동 노드를 실행하는 것과 동일 할 수 있습니다.``

There are some differences depending if you are connecting to a network running a genesis or bft consensus protocol.

``당신이 접속하는 네트워크(제네시스 또는 bft 합의 프로토콜)에 따라 약간의 차이가 있습니다.``

### Connecting to a genesis blockchain

#### Registering a stake pool

In order to be able to generate blocks in an existing genesis network, a [registered stake pool](../../stake_pool/registering_stake_pool) is needed.

``기존 제네시스 네트워크에서 블록을 생성할 수 있으려면 등록된 스테이크 풀이 필요합니다.``

#### Creating the secrets file

Put the node id and private keys in a yaml file in the following way:  

``다음과 같은 방법으로 노드 ID 와 개인 키를 yaml 파일에 넣습니다.``

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

``'asdf1234 ...' 부분은 네트워크의 실제 block0 해시 여야합니다.``

### Connecting to a BFT blockchain

In order to generate blocks, the node should be registered as a slot leader in the network and started in the following way.

``블록을 생성하려면 노드를 네트워크에서 슬롯 리더로 등록하고 다음과 같이 시작해야 합니다.``

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

``'asdf1234 ...' 부분은 네트워크의 실제 block0 해시 여야 합니다.``



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

- `format`: 로그 아웃풋 포맷 - `plain` 또는 `json`

- `output`: 로그 아웃풋 - `stdout`, `stderr`, `syslog` (Unix only),
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

``노드 구성 파일의 mempool 필드는 필수가 아니며 기본적으로 다음과 같이 설정됩니다.``

```yaml
mempool:
    fragment_ttl: 30m
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `fragment_ttl` 은 노드가 풀에서 프래그먼트(a _transaction_)를 얼마나 보관해야 하는지를 기술합니다.

* `log_ttl` 은 노드가 풀에서 보류/수락/거부 된 프래그먼트 로그를 보관하는 기간을 설명합니다.

* `garbage_collection_interval` 노드가 시간 초과된 항목(프래그먼트 또는 로그)을 제거 할 때의 간격을 설명합니다.


The `leadership` field in your node config file is not mandatory, by default it is set
as follow:

``노드 구성 파일의 리더십 필드는 필수는 아니며 기본적으로 다음과 같이 설정됩니다.``

```yaml
leadership:
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `log_ttl` 은 노드가 리더 이벤트 로그를 보관하는 기간을 기술합니다.

* `garbage_collection_interval` 은 2개의 가비지 수집 실행 간격 (예: 노드가 시간 초과된 item log 를 제거하는 경우)



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



# Staking with jormungandr

Here we will describe how to:

* _delegate your stake to a stake pool_ so that you can participate to the consensus
  and maybe collect rewards for that;

If you are a user of the blockchain then you will certainly not need to look
at the last section about _how to register a stake pool_.


``다음은 방법을 설명합니다.``

*  ``컨센서스에 참여하고 이에 대한 보상을 받을 수 있도록 스테이크 풀에 스테이크를 위임하십시오.``

``블록체인 사용자인 경우 스테이크 풀을 등록하는 방법에 대한 마지막 섹션을 반드시 볼 필요는 없습니다.``


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

* ``account public key : 공개 키의 bech32 문자열``
* ``the Stake Pool ID : 스테이크를 위임 할 스테이크 풀을 식별하는 16진수 문자열.``

```
$ jcli certificate new stake-delegation STAKE_POOL_ID ACCOUNT_PUBLIC_KEY > stake_delegation.cert
```

## how to sign your delegation certificate

We need to make sure that the owner of the account is authorizing this
delegation to happens, and for that we need a cryptographic signature.

``계정 소유자가 이 위임을 승인하고 암호화 서명이 필요한지 확인해야 합니다.``

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

``jcli transaction add-certificate 명령을 사용하여 트랜잭션을 완료하기 전에 인증서를 추가해야 합니다.``

For example:

```sh

...

jcli transaction add-certificate $(cat stake_delegation.cert) --staging tx

jcli transaction finalize CHANGE_ADDRESS --fee-constant 5 --fee-coefficient 2 --fee-certificate 2 --staging tx

...

```

The `--fee-certificate` flag indicates the cost of adding a certificate, used for computing the fees, it can be omitted if it is zero.

``--fee-certificate 플래그는 요금을 계산하는 데 사용되는 인증서 추가 비용을 나타내며 0 이면 생략할 수 있습니다.``

See [here](../jcli/transaction.md) for more documentation on transaction creation.

``트랜잭션 생성에 대한 자세한 내용은 여기를 참조하십시오.``


# registering stake pool

There are multiple components to be aware of when running a stake pool:

`` 스테이크 풀을 실행할 때 알아야 할 여러 구성 요소가 있습니다. ``

* your `NodeId`: 블록체인 프로토콜 내의 식별자입니다 (지갑은 이 NodeId를 통해 스테이크 풀에 위임합니다);
* your [**VRF**] key pairs: 리더 선거에 참여하기 위해 사용할 암호화 자료입니다;
* your [**KES**] key pairs: 블록에 서명하는 데 사용할 암호화 자료입니다.

So in order to start your stake pool you will need to generate these objects.

``따라서 스테이크 풀을 시작하려면 이러한 객체를 생성해야 합니다.``

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


``인증서는 귀하가 다른 블록 체인 참가자에게 자신을 스테이크 풀로 등록하기 위해 블록체인으로 전송하는 것입니다.``

```
$ jcli certificate new stake-pool-registration \
    --kes-key $(cat stake_pool_kes.pub) \
    --vrf-key $(cat stake_pool_vrf.pub) \
    --serial 1010101010 > stake_pool.cert
```

Now you need to sign this certificate with the owner key:

``이제 소유자 키로 이 인증서에 서명해야 합니다.``

```
$ cat stake_pool.cert | jcli certificate sign stake_key.prv | tee stake_pool.cert
cert1qsqqqqqqqqqqqqqqqqqqq0p5avfqp9tzusr26...cegxaz
```

And now you can retrieve your stake pool id (`NodeId`):

``이제 스테이크 풀 ID (NodeId)를 검색할 수 있습니다.``

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

``현재로서는 자체 블록체인 생성 구성을 만드는 방법에 대해서만 자세히 설명하지만 일반적인 경우 특정 블록체인 시스템에서 블록체인 구성을 사용할 수 있어야 합니다.``


# genesis file

The genesis file is the file that allows you to create a new blockchain
from block 0. It lays out the different parameter of your blockchain:
the initial utxo, the start time, the slot duration time, etc...

``생성 파일은 블록 0 에서 새 블록체인을 생성할 수 있는 파일입니다. 블록체인의 다른 매개 변수를 배치합니다 : 초기 utxo, 시작 시간, 슬롯 지속 시간 등.``

initial address UTxO 및 account UTxO 를 가진 BFT 생성 파일의 예.
BFT 블록체인 시작 및 주소에 관한 추가 정보 [starting a BFT blockchain here](./02_starting_bft_blockchain.md)
and [regarding addresses there](../jcli/address.md).
jcli 생성 명령어에 관한 정보도 찾을 수 있습니다. [jcli genesis tooling](../jcli/genesis.md).



You can generate a documented pre-generated genesis file:

``문서화된 사전 생성된 생성 파일을 생성할 수 있습니다.``

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

* `blockchain_configuration`: 이것은 블록 체인의 구성 파라미터 목록이며, 일부는 업데이트 프로토콜을 통해 나중에 변경될 수 있습니다.;
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
| `fund` | sequence | 블록체인의 초기 예치금 (항목 당 최대 255 개의 출력) |
| `cert` | string | initial certificate (초기 증명서) |
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

``legacy_fund 는 legacy Cardano 주소 형식만 다릅니다.``


# starting a bft node

BFT stands for the Byzantine Fault Tolerant
([read the paper](https://iohk.io/research/papers/#L5IHCV53)).

``BFT는 비잔틴 결함 허용을 나타냅니다([논문 읽기] (https://iohk.io/research/papers/#L5IHCV53))``

Jormungandr allows you to start a BFT blockchain fairly easily. The main
downside is that it is centralized, only a handful of nodes will ever have
the right to create blocks.

``Jormungandr를 사용하면 BFT 블록체인을 상당히 쉽게 시작할 수 있습니다. 주요 단점은 중앙 집중식이며 소수의 노드만 블록을 만들 수 있는 권리가 있다는 것입니다.``

## How does it work

It is fairly simple. A given number of Nodes (`N`) will generate
a key pairs of type `Ed25519` (see
[JCLI's Keys](./../jcli/key.md)).

``상당히 간단합니다. 주어진 수의 노드(`N`)는 `Ed25519` 유형의 키 쌍을 생성합니다. (참조 [JCLI 키] (./../jcli/key.md)).``

They all share the public key and add them in the genesis.yaml file.
It is the source of truth, the file that will generate the first block
of the blockchain: the **Block 0**.

``모두 공개 키를 공유하고 genesis.yaml 파일에 추가합니다. 블록체인의 첫 번째 블록인 블록 0 을 생성하는 것은 신뢰의 근본입니다.``

Then, only by one after the other, each Node will be allowed to create a block.
Utilising a Round Robin algorithm.

``그런 다음 하나씩 차례로 각 노드에 블록을 만들 수 있습니다. 라운드 로빈 알고리즘 활용.``

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

`` BFT 모드에서 블록체인을 시작하려면 다음을 확인해야 합니다. ``

* ``'consensus_leader_ids'는 비어 있지 않습니다.``

``[genesis file here] (./01_the_genesis_block.md)에 대한 자세한 정보``


## Creating the block 0

```
jcli genesis encode --input genesis.yaml --output block-0.bin
```

This command will create (or replace) the **Block 0** of the blockchain
from the given genesis configuration file (`genesis.yaml`).

``이 명령은 주어진 생성 구성 파일(`genesis.yaml`)에서 블록체인의 Block 0 을 생성(또는 대체)합니다.``

## Starting the node

Now that the blockchain is initialized, you need to start your node.

``이제 블록체인이 초기화되었으므로 노드를 시작해야 합니다.``

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

``1. 초기 설정 생성 `jcli genesis init> genesis.yaml` ``

``2. 비밀 키를 생성하십시오  ` jcli key generate --type=Ed25519 > key.prv` ``

``3. 비밀 키를 파일에 넣습니다  `node_secret.yaml은 다음과 같습니다 :` ``

```yaml
bft:
 signing_key: ed25519_sk1kppercsk06k03yk4qgea....
```

4. Generate public key out of previously generated key ` cat key.prv |  jcli key to-public`
5. Put generated public key as in `genesis.yaml` under `consensus_leader_ids:`
6. Generate block = `jcli genesis encode --input genesis.yaml --output block-0.bin`
7. Create config file and store it on your HD as `node.config` e.g. ->


``4. 퍼블릭키를 생성합니다. `cat key.prv | jcli key to public` ``

``5. 생성 된 공개 키를 `genesis.yaml` 파일에서 `consissus_leader_ids :` 에 입력합니다. ``

``6. 제네시스 블록 생성 = `jcli genesis encode --input genesis.yaml --output block-0.bin` ``

``7. 설정 파일을 만들어 당신의 HD에서  `node.config` 로 저장하십시오. -> ``

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

``또한 여기에는 bft 합의 프로토콜로 테스트 노드를 부트 스트랩하는 데 사용할 수 있는 스크립트가 있습니다.``

# starting a genesis blockchain

When starting a genesis praos blockchain there is an element to take
into consideration while constructing the block 0: _the stake distribution_.

``제네시스 praos 블록체인을 시작할 때 블록 0 을 구성하는 동안 고려해야 할 요소인 _the stake distribution_ 가 있습니다.``

In the context of Genesis/Praos the network is fully decentralized and it is
necessary to think ahead about initial stake pools and to make sure there
is stake delegated to these stake pools.

``Genesis/Praos의 context에서 네트워크는 완전히 분산되어 있으며 초기 스테이크 풀에 대해 미리 생각하고 스테이크 풀에 위임된 스테이크가 있는지 확인해야 합니다.``

In your genesis yaml file, make sure to set the following values to the appropriate
values/desired values:

``생성 yaml 파일에서, 다음 값을 적절한 값(원하는 값)으로 설정하십시오.``

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

``block0_consensus를 genesis_praos로 설정하면 기원 praos를 합의 레이어로 사용하여 블록체인을 시작하는 것입니다.``

`bft_slots_ratio` needs to be set to `0` (we don't support composite modes between
BFT mode and Genesis mode -- yet).

``bft_slots_ratio는 0으로 설정해야합니다 (BFT 모드와 Genesis 모드 사이의 복합 모드는 지원하지 않습니다).``

`consensus_genesis_praos_active_slot_coeff` determines minimum stake required to
try becoming slot leader, must be in range 0 exclusive and 1 inclusive.

``consensus_genesis_praos_active_slot_coeff는 슬롯 리더가 되기 위해 필요한 최소 스테이크를 결정합니다. 범위는 0과 1 사이 여야합니다.``

## The initial certificates

In the `initial_certs` field you will set the initial certificate. This is important
to declare the stake pool and delegate stake to them. Otherwise no block will be ever
created.

``initial_certs 필드에서 초기 인증서를 설정합니다. 스테이크 풀을 선언하고 스테이크를 위임하는 것이 중요합니다. 그렇지 않으면 블록이 생성되지 않습니다.``

Remember that in this array the **order** matters:

``이 배열에서 순서는 중요합니다.``

In order to delegate your stake, you need a stake pool to already exist, so the stake pool registration certificate should go first.

``스테이크를 위임하려면 스테이크 풀이 이미 존재해야하므로 스테이크 풀 등록 인증서가 먼저 있어야 합니다.``

### Stake pool registration

Now you can register a stake pool.
Follow the instruction in [registering stake pool guide](../stake_pool/registering_stake_pool.md).

``이제 스테이크 풀을 등록 할 수 있습니다. 스테이크 풀 가이드 등록 지침을 따르십시오.``

The _owner key_ (the key you sign the stake pool registration certificate) is the secret
key associated to a previously registered stake key.

``소유자 키(스테이크 풀 등록 인증서에 서명 한 키)는 이전에 등록된 스테이크 키와 관련된 비밀 키입니다.``

### Delegating stake

Now that there is both your stake key and there are stake pools available
in the block0 you need to delegate to one of the stake pool. Follow the instruction
in [delegating stake](../stake_pool/delegating_stake.md).

``이제 스테이크 키가 있고 블록 0 에 사용 가능한 스테이크 풀이 있으므로 스테이크 풀 중 하나에 위임해야 합니다. 스테이크 위임의 지시를 따르십시오.``

And in the initial funds start adding the addresses. To create an address with delegation
follow the instruction in [JCLI's address guide](../jcli/address.md). Utilise the stake key
registered previously as group address:

``그리고 초기 자금에서 주소 추가를 시작하십시오. 위임된 주소를 작성하려면 JCLI의 주소 안내서에 있는 지시 사항을 따르십시오. 이전에 그룹 주소로 등록된 스테이크 키를 사용하십시오.``

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
