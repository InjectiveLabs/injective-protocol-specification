---
description: >-
  This section is for advanced developers who are willing to integrate with our
  protocol directly.
---

# Testnet

Our testnet is a Tendermint based sidechain but abstracts away the necessity for users to send Tendermint style transactions. To perform actions on our sidechain \(e.g. creating orders, placing trades, canceling orders, etc.\) users can simply send HTTP requests to any relayer which runs our `relayer-api` REST server which is compliant with the [0x v3 Standard Relayer API specification](https://github.com/0xProject/standard-relayer-api/blob/master/http/v3.md).

The endpoints served by a Relayer API server are divided into the following subroutes:

* Relayer API `/api/sra/v3`— implements 0x SRAv3 mentioned above.
* REST API `/api/rest` — provides misc queries that are not present in SRA spec, used exclusively by our frontend clients and CLI tools.
* Coordinator API `/api/coordinator/v2` — implements the [0x v2 Coordinator specification](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/coordinator-specification.md).
* Derivatives API `/api/sda/v0` – our own standard similar to SRA, but adjusted for derivatives trading in DEX fashion.
* Chronos `/api/chronos/v1` – implements [TradingView UDF](https://github.com/tradingview/charting_library/wiki/UDF) provider endpoints as well as our own stats and history endpoints. Aggregates markets data and allows to filter own orders.

The full API specification can be found [here](https://api.injective.dev). OpenAPI YAML spec is located [here](https://api.injective.dev/openapi.yaml).

See [some examples](relayer-api-examples.md) on how to interact with the API endpoints.

## Testnet Sidechain Deployment

![](../../.gitbook/assets/screenshot-2020-07-30-at-12.35.57.png)

We run a small set of consensus validator nodes that are built using Cosmos SDK and operate using Tendermint protocol. While RPC of those nodes is not directly exposed, we run a covenient API wrapper - `relayer-api` that implements caching, data validation and adapts the responses to be compliant with SRA, TradingView, etc.

A few sentry or read-only nodes are replicating the state of the sidechain to different regions, and we set up an API endpoint for each supported region, so the frontentd and CLI tools can switch to the closest region with lower latency.

## Joining the Testnet

It is very easy for anyone to join the testnet and run own `relayerd` node in read-only mode. While it won't participate in consensus as a validator, it will be a full-node receiving all updates and keeping the state history. Such self-hosted node should be trusted 100% and you may run own `relayer-api` instance for your frontend of choice.

Further instructions for setting up a testnet sentry node are [here](testnet-join.md).

