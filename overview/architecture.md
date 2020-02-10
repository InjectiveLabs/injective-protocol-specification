# Architecture

The protocol is comprised of three principal components: 1\) the Injective sidechain relayer network, 2\) Injective's trade execution coordinator contract \(a smart contract on Ethereum\), and \(optionally\) 3\) a graphical front-end interface. Injective Protocol provides a fully decentralized sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator \(TEC\). 

![](../.gitbook/assets/architecture.png)

Injective Protocol uses an application-specific sidechain relayer network to maintain a decentralized orderbook and front-running resistant trade execution coordinator. 

We refer to our exchange model as **sidechain order relay with on-chain settlement** - a decentralized implementation of the traditionally centralized [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture) used by nearly all central limit order book decentralized exchanges. 

