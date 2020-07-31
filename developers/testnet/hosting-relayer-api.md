---
description: >-
  A guide on how to self-host an API server that exposes your relayerd sentry
  node for clients.
---

# Hosting Relayer API

## 1. Get docker image

The most straightforward way to get the latest version of `relayer-api` server distribution is to pull a Docker image. We use the exact same image for our deployments, so you will be up to date with the rest of the network. Learn [here](https://docs.docker.com/engine/install/ubuntu/) on how to install Docker itself.

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

This guide relies on Docker features, but is straightforward for binary distribution users as well.

## 2. Familiarize yourself with relayer-api options

{% code title="$ docker run -it --rm docker.injective.dev/core:latest relayer-api --help" %}
```text
Usage: relayer-api [OPTIONS] COMMAND [arg...]

Web server exposing injective-core API services.

Options:
      --env                   Application environment (env $APP_ENV) (default "local")
  -l, --log-level             Available levels: error, warn, info, debug. (env $APP_LOG_LEVEL) (default "info")
      --svc-wait-timeout      Standard wait timeout for all service dependencies (e.g. relayerd). (env $SERVICE_WAIT_TIMEOUT) (default "1m")
      --eth-node-ws           Specify WS endpoint for an Ethereum node. (env $ETHEREUM_RPC_WS) (default "ws://localhost:8545")
      --eth-node-http         Specify HTTP endpoint for an Ethereum node. (env $ETHEREUM_RPC_HTTP) (default "http://localhost:8545")
      --evm-node              Specify URI for an EVM sidechain node. (env $EVM_RPC_HTTP) (default "wss://evm-eu.injective.dev/ws")
      --zeroex-devutils       Ethereum address of 0x DevUtils contract. (Ex: 0xb125995F...) (env $ZEROEX_DEVUTILS_ADDR)
      --zeroex-exchange       Ethereum address of 0x Exchange contract. (Ex: 0xb125995F...) (env $ZEROEX_EXCHANGE_ADDR)
      --injective-futures     EVM address of Injective Futures contract. (Ex: 0xb125995F...) (env $INJECTIVE_FUTURES_ADDR)
  -B, --chronos-block         Specify block offset to watch fill events from. (env $CHRONOS_BLOCK_OFFSET) (default 0)
  -D, --chronos-data          Path to state DB of chronos server. (env $CHRONOS_DATA_PATH) (default "var/data")
      --relayerd-rpc-addr     Specify GRPC address of the relayerd service. (env $RELAYERD_RPC_ADDR) (default "localhost:9900")
      --tendermint-rpc-addr   Specify RPC address of a tendermint node. (env $TENDERMINT_RPC_ADDR) (default "tcp://localhost:26657")
      --http-listen-addr      HTTP server listening address (env $HTTP_LISTEN_ADDR) (default "localhost:4444")
      --ws-listen-addr        WebSocket server listening address (env $WS_LISTEN_ADDR) (default "localhost:4445")
      --statsd-prefix         Specify StatsD compatible metrics prefix. (env $STATSD_PREFIX) (default "relayer_api")
      --statsd-addr           UDP address of a StatsD compatible metrics aggregator. (env $STATSD_ADDR) (default "localhost:8125")
      --statsd-stuck-func     Sets a duration to consider a function to be stuck (e.g. in deadlock). (env $STATSD_STUCK_DUR) (default "5m")
      --statsd-mocking        If enabled replaces statsd client with a mock one that simply logs values. (env $STATSD_MOCKING) (default "false")
      --statsd-disabled       Force disabling statsd reporting completely. (env $STATSD_DISABLED) (default "false")

Commands:
  export                      Export Chronos DB as JSON

Run 'relayer-api COMMAND --help' for more information on a command.
```
{% endcode %}

Yes, there are many, but don't be afraid because most of them are static for all Testnet participants.

By default `relayer-api` keeps Chronos DB state in the `--chronos-data` directory which is `/apps/data/var/data` in the container. This can be overriden by CLI flag or ENV variables.

## 3. Prepare Docker Swarm config

Our example of deploying Relayer API server involves creating a Docker Swarm config `docker-compose.yml`that provides a very simple way to manage the service.

{% tabs %}
{% tab title="docker-compose.yml" %}
```yaml
version: "3.7"
services:
  relayer-api:
    image: docker.injective.dev/core:latest
    volumes:
       - chronos-data:/data/chronos
    deploy:
      replicas: 1
      endpoint_mode: vip
      restart_policy:
        condition: on-failure
    environment:
      - APP_ENV=dev
      - ETHEREUM_RPC_WS=wss://evm-us.injective.dev/ws
      - ETHEREUM_RPC_HTTP=https://evm-us.injective.dev
      - ZEROEX_DEVUTILS_ADDR=0xDf12200825b1D37F92a6A959813cd2B2bfA2c488
      - ZEROEX_EXCHANGE_ADDR=0xe525672f353e1b155f4495010D7814c5dd64a3C6
      - INJECTIVE_FUTURES_ADDR=0x8f399baf9009a1466d9a3d8372703427c9f0c8cc
      - CHRONOS_DATA_PATH=/data/chronos
      - RELAYERD_RPC_ADDR=172.17.0.1:9900
      - TENDERMINT_RPC_ADDR=tcp://172.17.0.1:26657
      - HTTP_LISTEN_ADDR=0.0.0.0:4444
      - WS_LISTEN_ADDR=0.0.0.0:4445
    command: relayer-api
    ports:
      - "8084:4444"
      - "8085:4445"
    networks:
      - relayer

networks:
  relayer:

volumes:
  chronos-data:

```
{% endtab %}

{% tab title="create.sh" %}
```bash
#!/bin/sh

# docker pull docker.injective.dev/core:latest
docker stack deploy --resolve-image=always -c docker-compose.yml relayer
docker stack ls
```
{% endtab %}

{% tab title="update.sh" %}
```bash
#!/bin/sh

docker pull docker.injective.dev/core:latest
docker service update --force relayer_relayer-api
docker service ls
```
{% endtab %}
{% endtabs %}

### EVM Endpoints available

* Europe `https://evm-eu.injective.dev` and `wss://evm-eu.injective.dev/ws`
* US `https://evm-us.injective.dev` and `wss://evm-us.injective.dev/ws`
* Asia \(Hong Kong\) `https://evm-ap.injective.dev` and `wss://evm-ap.injective.dev/ws`

Just pick one that is closer geographically to your sentry node and fill into the YAML template.

### Docker Swarm Init

One prerequisite is that Docker swarm must be active, either by joining to existing pool or by initialization on the current machine:

{% code title="$ docker swarm init" %}
```bash
Swarm initialized: current node (mniwbffn9913gq9mwe0r88qb5) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2tql4rttlmczysn8tavqrsr2u3f9n59rpn2qucj356de225udo-298f4i4zrjypqqv7ke6i7eus0 XXX.XXX.XXX.XXX:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```
{% endcode %}

### Preparing scripts

Create `create.sh` and `update.sh` scripts as suggested above to start relayer-api service. When creating the stack for the first time, run `create.sh`. The update script will download newer image and force-update the containers.

```text
$ chmod +x create.sh
$ chmod +x update.sh
```

## 4. Run API Server

Before starting Relayer API container, make sure that your Relayerd sentry node is fully synced and started its gRPC server.

Running the create script launches a new stack `relayer`:

{% code title="$ ./create.sh" %}
```bash
Creating network relayer_relayer
Creating service relayer_relayer-api
NAME                SERVICES            ORCHESTRATOR
relayer             1                   Swarm
```
{% endcode %}

To shutdown stack completely:

```text
$ docker stack rm relayer
```

Note that the Chronos DB volume survives the stack deletion. The volumes in replicated deployments are created on each swarm machine locally.

{% code title="$ docker volume list" %}
```text
DRIVER              VOLUME NAME
local               relayer_chronos-data
```
{% endcode %}

Listing running services in stack:

{% code title="$ docker service ls" %}
```text
ID                  NAME                  MODE                REPLICAS            IMAGE                              PORTS
n818lmaihmpx        relayer_relayer-api   replicated          1/1                 docker.injective.dev/core:latest   *:8084-8085->4444-4445/tcp
```
{% endcode %}

Reading Relayer API logs:

```text
$ docker service logs -f relayer_relayer-api
```

## 5. Expose Relayer API

If you are running the server using a stack config above, then API server exposes ports `8084` and `8085` for HTTP API and WS API accordingly. You might expose those using a reverse proxy such as Nginx or [Caddy](http://caddyserver.com). We use the following config for Caddy v2:

```text
https://testnet-api.injective.dev {
        reverse_proxy /api/sra/v3/ws 172.18.0.1:8085
        reverse_proxy 172.18.0.1:8084
}
```

The IP `172.17.0.1` in stack config corresponds to `docker0` allowing to access host ports from within a containter running in stack. And IP `172.18.0.1` there corresponds to `docker_gwbridge` allowing to access service ports from the host machine. Your environment and OS configuration might vary, so adjust accordingly.

