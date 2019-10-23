# Injective Protocol 0.1.0 Specification

## Table of contents

1.  [Architecture](#architecture)
1.  [Sidechain](#sidechain)
    1.  [Decentralized Orderbook](#decentralizedorderbook)
    1.  [Relayer API](#relayerapi)
    1.  [Trade Execution Coordinator](#tradeexecutioncoordinator)
1.  [Filter Contract](#filter)
1.  [Interface](#interface)
1.  [Governance](#governance)
1.  [Token Economics](#tokeneconomics)
1.  [Miscellaneous](#miscellaneous)
    1.  [Verifiable Delay Function](#verifiabledelayfunction)



# Architecture

Injective Protocol is a fully decentralized exchange protocol built on top of Ethereum and Cosmos. 

The protocol is comprised of three principal components: 1) the Injective sidechain relayer network, 2) Injective's filter contract (smart contract on Ethereum), and (optionally) 3) a front end interface. In this setup, the front end interface is used to communicate orders to and from the sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator (TEC). The sidechain relayer network aggregates trades in a canonical ordering (preventing front-running) and then submits the trades on Injective's [filter contract](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#filter-contracts) which in turn executes and settles the trades on 0x. 



<img alt="diagram-injective.png" src="https://cl.ly/f9077bd91d24/download/diagram-injective.png" width="700px"/>

# Sidechain

Injective Protocol uses an application-specific sidechain relayer network to maintain a decentralized orderbook, front-running resistant trade execution coordinator, and order matching and execution engine. We refer to our model as **sidechain order relay with on-chain settlement** - a decentralized implementation of the traditionally centralized [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture) used by nearly all central limit order book decentralized exchanges. The sidechain is built on top of Tendermint and the application logic is implemented using the Cosmos SDK. 

### Decentralized Orderbook

The Injective sidechain hosts a decentralized, censorship-resistant orderbook which stores and relays orders. 

#### Make Order Relay Procedure

1. A valid signed 0x order is first created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer performs validation on the order and checks
	1. The order format integrity
		1. The order should conform to the [0x order message format](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) 
		2. The order should be [`fillable`](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#getorderinfo) 
		3. The `feeRecipientAddress` must match the originating relayer's Ethereum address in the relayer address registry. 
		4. The `takerAddress` must match the address of the Injective Filter Contract
		5. The `senderAddress` must match the address of the Injective Filter Contract
	2. The maker's  `makerAssetData` balance should be greater than or equal to the `makerAssetAmount`
	2. The maker's  `makerAssetData`  approval amount should be greater than or equal to the `makerAssetAmount` of the order
	3. The signature validity for the hash of the order
	4. The trading pair is supported by the sidechain
4. If the order is valid, the relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the make order to its peers in the sidechain network
5. Each sidechain node calls [CheckTx](https://tendermint.com/docs/app-dev/abci-spec.html#checktx) on the transaction which performs [standard Tendermint checks](https://github.com/cosmos/cosmos-sdk/blob/master/docs/basics/tx-lifecycle.md#addition-to-mempool) on the transaction and adds/discards the transaction to the mempool.
6. Nodes reach consensus on on which transactions to include in each round through [Tendermint BFT](https://tendermint.com/docs/spec/consensus/consensus.html) and commit to a new block, fully executing the state transitions from each transaction.
   1. Each make order message (transaction) is routed to the `handleMsgCreateMakeOrder` handler function which stores it in the keeper provided the order and trade pair are valid

### Relayer API
Each relayer can optionally support their own API allowing for 1) submission of orders to the sidechain and 2) query of data of the application state. The following implementation is provided out-of-the-box for relayers to use, but each relayer is free to provide their own API for their desired use case. 
