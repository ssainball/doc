# Quickstart

The rust node comes with tools and help in order to quickly start
a node and connect to the blockchain.

It is compatible with most platforms and it is pre-packaged for some
of them.

Here we will see how to install `jormungandr` and its helper `jcli`
and how to connect quickly to a given blockchain.

There are three posible ways you can start jormungandr.

## As a passive node in an existing network

As described [here](./node_types/01_passive_node.md).  

The passive Node is the most common type of Node on the network. It can be used to download the blocks and broadcast transactions to peers, but it
doesn't have cryptographic materials or any mean to create blocks.
This type of nodes are mostly used for wallets, explorers or relays.

## As a node generating blocks in an existing network

The network could be running either bft or genesis consensus. In the former case the node must have the private key of a registered as a slot leader, while for the latter the private keys of a registered stake pool are needed. 

More information [here](./node_types/02_generating_blocks.md)

## Creating your own network

This is similar to the previous case, but configuring a genesis file is needed. Consult the [Advanced section](../advanced/introduction.md) for more information on this procedure.



The software is bundled with 2 different command line software:

1. **jormungandr**: the node;
2. **jcli**: Jormungandr Command Line Interface, the helpers and primitives to run and interact with the node.

# Installation

## From a release

This is the recommended method. Releases are all available
[here](https://github.com/input-output-hk/jormungandr/releases).

## From source

Jormungandr's code source is available on
[github](https://github.com/input-output-hk/jormungandr#how-to-install-from-sources).
Follow the instructions to build the software from sources.

# Help and auto completion

All commands come with usage help with the option `--help` or `-h`.

For `jcli`, it is possible to generate the auto completion with:

```
jcli auto-completion bash ${HOME}/.bash_completion.d
```

Supported shells are: bash, fish, zsh, powershell and elvish.

**Note:** 
Make sure `${HOME}/.bash_completion.d` directory previously exists on your HD.
In order to use auto completion you still need to:
```
source ${HOME}/.bash_completion.d/jcli.bash
```
You can also put it in your `${HOME}/.bashrc`.



In order to start the node, you first need to gather the blockchain
information you need to connect to.

1. the hash of the **genesis block** of the blockchain, this will be the source
   of truth of the blockchain. It is 64 hexadecimal characters.
2. the **trusted peers** identifiers and access points.

These information are essentials to start your node in a secure way.

The **genesis block** is the first block of the blockchain. It contains the
static parameters of the blockchain as well as the initial funds. Your node
will utilise the **Hash** to retrieve it from the other peers. It will also
allows the Node to verify the integrity of the downloaded **genesis block**.

The **trusted peers** are the nodes in the public network that your Node will
trust in order to initialise the Peer To Peer network.

# The node configuration

Your node configuration file may look like the following:

**Note**

This config shouldn't work as it is, the ip address and port for the trusted peer should be those of an already running node.
Also, the public_address ('u.x.v.t') should be a valid address (you can use an internal one, eg: 127.0.0.1).
Furthermore, you need to have permission to write in the path specified by the storage config.

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

# Starting the node

```
jormungandr --config config.yaml --genesis-block-hash 'abcdef987654321....'
```

The 'abcdef987654321....' part refers to the hash of the genesis, that should be given to you from one of the peers in the network you are connecting to.

In case you have the genesis file (for example, because you are creating the network) you can get this hash with jcli.

```sh
cat block-0 | jcli genesis hash
```

or, in case you only have the yaml file

```sh
cat genesis.yaml | jcli genesis encode | jcli genesis hash
```

[`Ed25519`]: ../jcli/key.md



It is possible to query the node via its REST Interface.

In the node configuration, you have set something like:

```yaml
# ...

rest:
  listen: "127.0.0.1:8443"

#...
```

This is the REST endpoint to talk to the node, to query blocks or send transaction.

It is possible to query the node stats with the following end point:

```
curl http://127.0.0.1:8443/api/v0/node/stats
```

The result may be:

```json
{"blockRecvCnt":120,"txRecvCnt":92,"uptime":245}
```

> THE REST API IS STILL UNDER DEVELOPMENT

Please note that the end points and the results may change in the future.

To see the whole Node API documentation,
[click here](https://editor.swagger.io/?url=https://raw.githubusercontent.com/input-output-hk/jormungandr/master/doc/openapi.yaml)



# Explorer mode

The node can be configured to work as a explorer. This consumes more resources, but makes it possible to query data otherwise not available.

## Configuration

There is two ways of enabling the explorer api. It can either be done by passing the `--enable-explorer` flag on the start arguemnts or by the config file: 

``` yaml
explorer:
    enabled: true
```

#### CORS

For configuring CORS the explorer API, this needs to be done on the REST section of the config, as documented [here](../configuration/network.md).

## API

A graphql interface can be used to query the explorer data, when enabled, two endpoints are available in the [REST interface](03_rest_api.md): `/explorer/graphql` and `/explorer/graphiql` .

The first is the one that queries are made against, for example: 

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



# How to start a node as a leader candidate

## Gathering data

Like in the passive node case, two things are needed to connect to an existing network

1. the hash of the **genesis block** of the blockchain, this will be the source
   of truth of the blockchain. It is 64 hexadecimal characters.
2. the **trusted peers** identifiers and access points.

The node configuration could be the same as that for [running a passive node](./01_passive_node.md). 

There are some differences depending if you are connecting to a network running a genesis or bft consensus protocol.

### Connecting to a genesis blockchain

#### Registering a stake pool

In order to be able to generate blocks in an existing genesis network, a [registered stake pool](../../stake_pool/registering_stake_pool) is needed.

#### Creating the secrets file

Put the node id and private keys in a yaml file in the following way:  

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

### Connecting to a BFT blockchain

In order to generate blocks, the node should be registered as a slot leader in the network and started in the following way.

## The secret file

Put secret key in a yaml file, e.g. `node_secret.yaml` as follows:

```yaml
bft:
 signing_key: ed25519_sk1kppercsk06k03yk4qgea....
```

where signing_key is a private key associated to the public id of a slot leader.

### Starting the node

```sh
jormungandr --genesis-block asdf1234... --config node.config --secret node_secret.yaml
```

_The 'asdf1234...' part should be the actual block0 hash of the network_
