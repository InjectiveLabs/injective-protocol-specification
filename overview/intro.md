---
description: >-
  Injective Protocol is a fully decentralized layer-2 decentralized exchange
  protocol supporting the full potential of borderless decentralized finance.
---

# Introduction

### Background

Currently, existing decentralized exchanges are unable to provide secure and fast matching without having a centralized component throughout the order matching and settlement process. Popular spot trading DEXs like IDEX, DDEX, RadarRelay, and Etherdelta all have a centralized server which hosts the orderbook and facilitates order matching. Not only are such solutions centralized, but they also are prone to front-running and have limited throughput due to every transaction having to occur on-chain on Ethereum. 

### About

Injective Protocol solves the scalability and security issues faced by decentralized exchanges with a fully-decentralized, open, permissionless decentralized exchange protocol that turns exchange into a public utility in which anyone can participate and capture value in. Orders are matched on a layer-2 Cosmos [sidechain](intro.md#sidechain) and settled on 0x via a decentralized [trade execution coordinator](intro.md#trade-execution-coordination) contract. The protocol integrates [verifiable delay functions](intro.md#verifiable-delay-function) to prevent front-running and order collisions, allowing users to submit orders securely.

The exchange protocol conforms to 0x on Ethereum and can be easily integrated with Ethereum, Cosmos, and Binance Chain DeFi ecosystems. However, unlike a standard 0x open orderbook model, users do not need to pay for Ethereum gas or exchange fees in ZRX. Injective also supports advanced exchange features such as [soft cancellations](intro.md#soft-cancel-creation) and selective delay by default.

The protocol's [native token](intro.md#token-economics) will be used to maintain proof-of-stake security on the sidechain, reward order discovery and origination for nodes, and allow token holders to capture value on the success of the protocol via a token burn or distribution mechanism. The exchange protocol does not collect fees in native token by default but rather implements negative spread, thus following the fee model of traditional centralized exchanges. The fees collected will undergo a periodic auction enforced on a smart contract to buy back the native token.

