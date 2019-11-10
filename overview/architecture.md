# Architecture

The protocol is comprised of three principal components: 1\) the Injective sidechain relayer network, 2\) Injective's trade execution coordinator contract \(a smart contract on Ethereum\), and \(optionally\) 3\) a graphical front-end interface. Injective Protocol provides a fully decentralized sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator \(TEC\). Individuals can communicate orders/trades \(e.g. through a front-end interface\) to the sidechain relayer network which aggregates trades in a canonical manner which prevents front-running and then submits the on Injective's trade execution [coordinator contract](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#filter-contracts) which in turn executes and settles the trades through 0x.

![architecture.png](https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/.gitbook/assets/architecture.png?raw=true)

Injective Protocol uses an application-specific sidechain relayer network to maintain a decentralized orderbook, front-running resistant trade execution coordinator, and order matching and execution engine.

We refer to our exchange model as **sidechain order relay with on-chain settlement** - a decentralized implementation of the traditionally centralized [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture) used by nearly all central limit order book decentralized exchanges.

