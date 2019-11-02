This chapter covers the node documentation, necessary to have a working system. It covers
the network, logging and storage parameters.

The node configuration uses the [YAML](https://en.wikipedia.org/wiki/YAML) format.

This is an example of a configuration file:

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

- `level`: log messages minimum severity. If not configured anywhere, defaults to "info".
  Possible values: "off", "critical", "error", "warn", "info", "debug", "trace".
- `format`: log output format - `plain` or `json`.
- `output`: log output - `stdout`, `stderr`, `syslog` (Unix only),
  or `journald` (Linux with systemd only, must be enabled during compilation).


There's 2 differents network interfaces which are covered by their respective section:

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

The `mempool` field in your node config file is not mandatory, by default it is set
as follow:

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


The `leadership` field in your node config file is not mandatory, by default it is set
as follow:

```yaml
leadership:
    log_ttl: 1h
    garbage_collection_interval: 15m
```

* `log_ttl` describes for how long the node will keep logs of leader events.
  This is link to the data you receives from the REST leadership logs end point;
* `garbage_collection_interval` describes the interval between 2 garbage collection
  runs: i.e. when the node removes item logs that have timed out
