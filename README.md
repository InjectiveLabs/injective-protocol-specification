# Injective Protocol 0.1.0 Specification

## Table of contents

1.  [Overview](#overview)
1.  [Architecture](#architecture)
1.  [Sidechain](#sidechain)
    1.  [Decentralized Orderbook](#decentralized-orderbook)  
        1. [Make Order Creation](#make-order-creation)
        1. [Take Order Creation](#take-order-creation)
        1. [Make Order Procedure](#make-order-procedure)
        1. [Make Order Procedure](#make-order-procedure)
    1.  [Relayer API](#relayer-api)
    1.  [Trade Execution Coordinator](#trade-execution-coordinator)
1.  [Filter Contract](#filter)
1.  [Interface](#interface)
1.  [Governance](#governance)
1.  [Token Economics](#token-economics)
1.  [Miscellaneous](#miscellaneous)
    1.  [Verifiable Delay Function](#verifiable-delay-function)


# Overview
Injective Protocol is a fully decentralized exchange protocol built on top of Ethereum and Cosmos. 

* describe problem
* fully-decentralized, open, permissionless
* front-running proof
* native token
* 0x trades, extensible to arbitrary DeFi
* gasless transactions

# Architecture
The protocol is comprised of three principal components: 1) the Injective sidechain relayer network, 2) Injective's filter contract (smart contract on Ethereum), and (optionally) 3) a front end interface. In this setup, the front end interface is used to communicate orders to and from the sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator (TEC). The sidechain relayer network aggregates trades in a canonical ordering (preventing front-running) and then submits the trades on Injective's [filter contract](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#filter-contracts) which in turn executes and settles the trades on 0x. 

<img alt="diagram-injective.png" src="https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/assets/architecture.png" width="700px"/>

# Sidechain

Injective Protocol uses an application-specific sidechain relayer network to maintain a decentralized orderbook, front-running resistant trade execution coordinator, and order matching and execution engine. We refer to our model as **sidechain order relay with on-chain settlement** - a decentralized implementation of the traditionally centralized [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture) used by nearly all central limit order book decentralized exchanges. The sidechain is built on top of Tendermint and the application logic is implemented using the Cosmos SDK. 

## Decentralized Orderbook

The Injective sidechain hosts a decentralized, censorship-resistant orderbook which stores and relays orders. The application logic for the orderbook is in the [`orders`](https://github.com/InjectiveLabs/injective-core/tree/master/cosmos/x/orders) module which supports six distinct actions (i.e. state transitions in the form of [Msgs](https://godoc.org/github.com/cosmos/cosmos-sdk/types#Msg)):

* Make order creation
* Take order creation
* Soft cancel 
* Trading pair creation
* Trading pair suspension
* Trading pair resumption

The procedure involved with each action is described below, with each procedure for the above being followed by the [Tendermint procedure](#tendermint-procedure) (which we briefly include for completeness of understanding). Further documentation on the orders module and Tendermint can be found [here](https://github.com/InjectiveLabs/injective-core/blob/master/cosmos/x/orders/README.md) and [here](https://tendermint.com/docs/introduction/) respectively. 

### Make Order Creation
1. A valid signed make order is created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer performs validation on the order and checks that:
	1. The order format integrity is kept
		1. The order should conform to the [0x order message format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) 
		2. The order should be [`fillable`](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#getorderinfo) 
		3. The `feeRecipientAddress` must match the originating relayer's Ethereum address in the relayer address registry
		4. The `takerAddress` must match the address of the Injective Filter Contract
		5. The `senderAddress` must match the address of the Injective Filter Contract
	2. The maker's  `makerAssetData` balance should be greater than or equal to the `makerAssetAmount`
	2. The maker's  `makerAssetData`  approval amount should be greater than or equal to the `makerAssetAmount` of the order
	3. The signature validity for the hash of the order
	4. The trading pair is supported by the sidechain
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network (assuming step 3 passes)
5. The make order msg is handled and added to the orderbook

### Take Order Creation
1. A valid signed take order is created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer performs the same validation of the order as in the make order prodedure and also checks that:
	1. The make order fill amounts are fillable 
	2. The trade conforms to the negative spread model
	3. `VdfInput` of the order hash is valid (if supplied)
	4. `VdfOutput` is the valid output for the VDF applied on `VdfInput` for `VdfIterations` (if supplied)
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network (assuming step 3 passes)
5. The take order msg is handled and added to the pending queue of take orders to be submitted for that block

### Soft Cancel 
1. A valid signed cancel order is created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer validates the cancel order 
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network (assuming step 3 passes)
5. The make order in question is marked as soft-cancelled and take orders in the pending queue which include the soft-cancelled make order are removed. 

### Trading Pair Creation 
TBD

### Trading Pair Suspension
TBD

### Trading Pair Resumption
TBD

### Tendermint Procedure
1. Each sidechain node calls [CheckTx](https://tendermint.com/docs/app-dev/abci-spec.html#checktx) on the transaction which performs [standard Tendermint checks](https://github.com/cosmos/cosmos-sdk/blob/master/docs/basics/tx-lifecycle.md#addition-to-mempool) on the transaction and adds/discards the transaction to the mempool.
2. Nodes reach consensus on on which transactions to include in each round through [Tendermint BFT](https://tendermint.com/docs/spec/consensus/consensus.html) and commit to a new block, fully executing the state transitions from each transaction.
   1. Each order message (transaction) is routed to its designated handler function which performs further checks and executes changes to the application store. 


## Relayer API
Each relayer can optionally support their own API allowing for 1) submission of orders to the sidechain and 2) query of data of the application state. The following implementation is provided out-of-the-box for relayers to use, but each relayer is free to provide their own API for their desired use case. 
