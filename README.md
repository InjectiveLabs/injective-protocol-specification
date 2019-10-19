# injective-protocol-specification
Specification for Injective Protocol

## Table of contents

1.  [Architecture](#architecture)
1.  [Sidechain](#sidechain)
      1.  [Decentralized Orderbook](#decentralizedorderbook)
      1.  [Relayer](#relayer)
1.  [Filter Contract](#filter)



# Architecture

Injective Protocol is a fully decentralized exchange protocol built on top of Ethereum and Cosmos. 

The protocol is comprised of three principal components: 1) the Injective sidechain relayer network, 2) Injective's filter contract (smart contract on Ethereum), and (optionally) 3) a front end interface. In this setup, the front end interface is used to communicate orders to and from the sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator (TEC). The sidechain relayer network aggregates trades in a canonical ordering (preventing front-running) and then submits the trades on Injective's [filter contract](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#filter-contracts) which in turn executes and settles the trades on 0x. 



<img alt="diagram-injective.png" src="https://cl.ly/f9077bd91d24/download/diagram-injective.png" width="700px"/>

# Sidechain

The sidechain relayer network uses a novel approach we refer to as **sidechain order relay with on-chain settlement** - a decentralized implementation of [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture). 


