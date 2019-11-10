## Introduction
Injective Protocol is a fully decentralized exchange protocol built on top of Ethereum and Cosmos. 

Currently, there are very few fully-decentralized exchanges that delivers secure and fast matching without having a centralized component throughout the order matching and settlement process. Popular spot trading DEXs like IDEX, DDEX, RadarRelay, and Etherdelta all have a centralized server maintaining the orderbook and facilitating order matching. Many exchanges like Etherdelta, Oasis, and Eth2Dai are also vulnerable to front-running, with 0x team citing a order failure rate of up to 21% on Etherdelta. 

Injective Protocol sets out to solve these issues by creating a fully-decentralized, open, permissionless decentralized exchange protocol that turns exchange into a public utility where everyone can participate and capture value in. Orders are matched on a layer-2 Cosmos [sidechain](#sidechain) and settled on 0x via [trade execution coordinator](#trade-execution-coordination) contract. The protocol integrates [verifiable delay function](#verifiable-delay-function) to combat front-running and order collision, allowing users to submit orders safely without worrying about a long and probabilistic order execution,

The exchange protocol implements 0x order schema on Ethereum and can be easily integrated with Ethereum, Cosmos, and Binance Chain DeFi ecosystems. However, unlike a standard 0x open orderbook model, users do not need to pay for Ethereum gas or exchange fees in ZRX. Injective also supports advanced exchange features such as [soft cancel](#soft-cancel-creation) and selective delay by default.

The protocol's [native token](#token-economics) will be used to maintain proof-of-stake security on the sidechain, reward order discovery and origination for nodes, and allow token holders to capture value on the success of the protocol via a token burn or distribution mechanism. The exchange protocol does not collect fees in native token by default but rather implements a negative spread model like most of the traditional centralized exchanges. The fees collected will undergo a periodic auction enforced on a smart contract to buy back the native token.


<!-- * describe problem
* fully-decentralized, open, permissionless
* front-running proof
* native token
* 0x trades, extensible to arbitrary DeFi
* gasless transactions -->