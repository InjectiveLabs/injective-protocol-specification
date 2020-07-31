---
description: >-
  The shortest guide on how to host own relayerd sentry node and join the
  Injective Testnet.
---

# Joining the Testnet

## 1. Get docker image

The most straightforward way to get the latest version of `relayerd` and `relayer-api` distribution is to pull a Docker image. We use the exact same image for our deployments, so you will be up to date with the rest of the network. Learn [here](https://docs.docker.com/engine/install/ubuntu/) on how to install Docker itself.

```
$ docker pull docker.injective.dev/core:latest

$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
docker.injective.dev/core   latest              99e2d2457df0        44 hours ago        145MB
docker.injective.dev/core   <none>              8c951f6c8274        4 days ago          145MB
```

{% hint style="info" %}
 Installation by compiling from source is not available yet, as we are preparing a public code release in Q3 2020. Stay tuned for updates in our Telegram [group](https://t.me/joininjective).
{% endhint %}

If you are not a big fan of managing Docker images, stacks and want to avoid extra parameters in the commands, you can fetch a pre-built binaries for your OS here: [https://github.com/InjectiveLabs/injective-core-releases/releases/](https://github.com/InjectiveLabs/injective-core-releases/releases/)

## 2. Familiarize yourself with Relayerd options

{% code title="$ docker run -it --rm docker.injective.dev/core:latest relayerd --help" %}
```text
Relayer Daemon (a Tendermint node)

Usage:
  relayerd [command]

Available Commands:
  init                Initialize private validator, p2p, genesis, and application configuration files
  collect-gentxs      Collect genesis txs and output a genesis.json file
  migrate             Migrate genesis to a specified target version
  gentx               Generate a genesis tx carrying a self delegation
  validate-genesis    validates the genesis file at the default location or at the location passed as an arg
  add-genesis-account Add a genesis account to genesis.json

  keys                Add or view local private keys
  debug               Tool for helping with debugging your application
  info                Print version info
  start               Run the full node
  unsafe-reset-all    Resets the blockchain database, removes address book files, and resets priv_validator.json to the genesis state

  tendermint          Tendermint subcommands
  export              Export state to JSON

  version             Print the app version
  help                Help about any command

Flags:
  -b, --broadcast-mode string        Transaction broadcasting mode (sync|async|block) (default "sync")
      --chain-id string              Specify Chain ID for sending Tx (default "testnet")
      --eth-coordinator string       Ethereum address of Injective 0x Coordinator contract. (Ex: 0xb125995F...)
      --eth-from string              Ethereum wallet address to send from. (Ex: 0xf91fb157...)
      --eth-from-passphrase string   Passphrase for wallet private key specified with 'from' (default "12345678")
      --eth-from-pk string           Ethereum wallet private key (for testing only) (Ex: 5D862464FE95...)
      --eth-futures string           Ethereum address of Injective Futures contract. (Ex: 0xb125995F...)
      --eth-node-http string         HTTP endpoint for an Ethereum node. (default "http://localhost:8545")
      --eth-node-ws string           WebSocket endpoint for an Ethereum node. (default "ws://localhost:8545")
      --fees string                  Fees to pay along with transaction; eg: 10uatom
      --from string                  Name or address of private key with which to sign
      --from-passphrase string       Passphrase for private key specified with 'from' (default "12345678")
      --gas string                   gas limit to set per-transaction; set to "auto" to calculate required gas automatically (default 200000) (default "200000")
      --gas-adjustment float         adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)
      --gas-prices string            Gas prices to determine the transaction fee (e.g. 10uatom)
      --grpc-listen-addr string      gRPC server listening address (default "localhost:9900")
  -h, --help                         help for relayerd
      --home string                  directory for config and data (default "/root/.relayerd")
      --keyring-backend string       Select keyring's backend (default "file")
      --log_level string             Log level (default "main:info,state:info,*:error")
      --node string                  <host>:<port> to tendermint rpc interface for this chain (default "tcp://localhost:26657")
      --statsd-address string        UDP address of a StatsD compatible metrics aggregator. (default "localhost:8125")
      --statsd-enabled               Enabled StatsD reporting.
      --statsd-prefix string         Specify StatsD compatible metrics prefix. (default "relayerd")
      --statsd-stuck-func string     Sets a duration to consider a function to be stuck (e.g. in deadlock). (default "5m")
      --trace                        print out full stack trace on errors
      --trust-node                   Trust connected full node (don't verify proofs for responses) (default true)
      --zeroex-devutils string       Ethereum address of 0x DevUtils contract. (Ex: 0xb125995F...)
      --zeroex-exchange string       Ethereum address of 0x Exchange contract. (Ex: 0xb125995F...)

Use "relayerd [command] --help" for more information about a command.
```
{% endcode %}

Yes, there are many, but don't be afraid because most of them are static for all Testnet participants.

By default `relayerd` keeps all state in the `--home` directory which is `/root/.relayerd` in the container. So you need to specify a volume mount from the host directory into container. That way the state will survive during restarts of the Docker container.

```bash
$ mkdir ~/relayerd
$ alias relayerd='docker run -it -v ~/relayerd:/root/.relayerd --rm docker.injective.dev/core relayerd'
# ^ put that alias into .bashrc or something
```

## 3. Init Relayerd state

Before syncing up with the rest of the testnet, you need to init a full node state. Right after init, the genesis config needs to be replaced with our pre-baked version. So your node will catch up with others on the first start.

```bash
$ relayerd init [your_custom_moniker] --chain-id testnet
```

You can edit this node moniker later, in the `$HOME/relayerd/config/config.toml` file. When running without Docker, the default location would be `$HOME/.relayerd`, and with Docker the state dir would be owned by root — so use `sudo` to edit this.

```bash
# A custom human readable name for this node
moniker = "<your_custom_moniker>"
```

You must also set the list of persistent peers so your node can sync the state from these bootstrap nodes.

```bash
# Comma separated list of nodes to keep persistent connections to
persistent_peers = "e58854fcaf218ae0f667bf5750eef467d357d13f@testnet-rpc.injective.dev:26656,07d0159602fca092c4702f514e994879ab78064e@testnet-rpc-us.injective.dev:26656,3816d76550646ca74d57b2ed16ff1acc2b7925f0@testnet-rpc-ap.injective.dev:26656"
```

Optionally enable Prometheus metrics for the node.

```bash
# When true, Prometheus metrics are served under /metrics on
# PrometheusListenAddr.
# Check out the documentation for the list of available metrics.
prometheus = true

# Address to listen for Prometheus collector(s) connections
prometheus_listen_addr = ":26660"
```

Save and close the `config.toml` file.

Download the latest [genesis.json](https://testnet-genesis.injective.dev/genesis.json) for the Testnet network:

```text
$ wget https://testnet-genesis.injective.dev/genesis.json -O $HOME/relayerd/config/genesis.json
```

## 4. Prepare launch script

To launch relayerd for syncing you'll need to set some parameters to their defaults for Testnet. Just copy this launch script template and place it into `$HOME/relayerd-testnet.sh`:

{% code title="relayerd-testnet.sh" %}
```bash
#!/bin/sh

# NOTE: chain-id in relayerd arguments is Cosmos chain.
# Both Ethereum Network ID and Chain ID are fetched from the node.

docker run -it --rm \
	-v ~/relayerd:/root/.relayerd \
	-p 26657:26657 -p 9900:9900 \
	docker.injective.dev/core relayerd \
	--chain-id=testnet \
	--eth-coordinator="0x3b46eF40b11888b7353C764Fca86A83fF89dC90C" \
	--eth-futures="0x8f399baf9009a1466d9a3d8372703427c9f0c8cc" \
	--zeroex-devutils="0x2C09F049f620bF04390E146b6CD7534311614a29" \
	--zeroex-exchange="0xe525672f353e1b155f4495010D7814c5dd64a3C6" \
	--eth-node-ws="wss://evm-eu.injective.dev/ws" \
	--eth-node-http="https://evm-eu.injective.dev" \
	--log_level="*:info" \
	--rpc.laddr="tcp://0.0.0.0:26657" \
	--grpc-listen-addr "0.0.0.0:9900" \
	start
```
{% endcode %}

### EVM Endpoints available

* Europe `https://evm-eu.injective.dev` and `wss://evm-eu.injective.dev/ws`
* US `https://evm-us.injective.dev` and `wss://evm-us.injective.dev/ws`
* Asia \(Hong Kong\) `https://evm-ap.injective.dev` and `wss://evm-ap.injective.dev/ws`

Just pick one that is closer geographically to your sentry node and fill into the script template.

One last step is to make the script executable:

```bash
$ chmod +x relayerd-testnet.sh
```

## 5. Sync the node

Run the script to start syncing the node with the rest of Testnet. You should not close terminal, as this script is not daemonized — if you replace `-it --rm` with `-d` in the command to run a detached container. There is a lot of ways to daemonize this alternatively — using `systemd`, `docker-compose` or even `docker stack`. Daemonization of the relayerd process is out of scope of this tutorial.

```text
$ ./relayer-testnet.sh

...
I[2020-07-29|18:32:07.878] No 'from' account provided, loopback client will be in read-only mode module=main
I[2020-07-29|18:32:07.891] RelayerDaemon init done                      module=main.
...
```

Note that the node without `--from` argument will run in read-only mode and won't be able to submit sidechain transactions, e.g. when posting the order. Please contact with Injective Team to get some Testnet coins to pay for transaction gas. A simple restart would be enough to activate transaction sending then.

The next step after the node is fully synced would be to launch own Relayer API gateway.

