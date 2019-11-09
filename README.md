# Injective Protocol 1.0.0 Specification (WORK IN PROGRESS)

## Table of contents

1.  [Overview](#overview)
1.  [Architecture](#architecture)
1.  [Sidechain](#sidechain)
    1.  [Decentralized Orderbook](#decentralized-orderbook)  
        1. [Make Order Creation](#make-order-creation)
        1. [Take Order Creation](#take-order-creation)
        1. [Soft Cancel Creation](#soft-cancel-creation)
        1. [Trading Pair Creation](#trading-pair-creation)
        1. [Trading Pair Suspension](#trading-pair-suspension)
        1. [Trading Pair Resumption](#trading-pair-resumption)
        1. [Tendermint Procedure](#tendermint-procedure)
    1.  [Relayer API](#relayer-api)
    1.  [Trade Execution Coordination](#trade-execution-coordination)
    1.  [Validator Requirements](#validator-requirements)
        1. [Registration and Stake](#registration-and-stake)
        1. [Validator Duties](#validator-duties)
        1. [Validator Rewards](#validator-rewards)
        1. [Slashing](#slashing)
1.  [Coordinator Contract](#coordinator-contract)
1.  [Interface](#interface)
1.  [Governance](#governance)
1.  [Token Economics](#token-economics)
1.  [Miscellaneous](#miscellaneous)
    1.  [Verifiable Delay Function](#verifiable-delay-function)


## Overview
Injective Protocol is a fully decentralized exchange protocol built on top of Ethereum and Cosmos. 

Currently, there are very few fully-decentralized exchanges that delivers secure and fast matching without having a centralized component throughout the order matching and settlement process. Popular spot trading DEXs like IDEX, DDEX, RadarRelay, and Etherdelta all have a centralized server maintaining the orderbook and facilitating order matching. Many exchanges like Etherdelta, Oasis, and Eth2Dai are also vulnerable to front-running, with 0x team citing a order failure rate of up to 21% on Etherdelta. 

Injective Protocol sets out to solve these issues by creating a fully-decentralized, open, permissionless decentralized exchange protocol that turns exchange into a public utility where everyone can participate and capture value in. Orders are matched on a layer-2 Cosmos [sidechain](#sidechain) and settled on 0x via [trade execution coordinator](#trade-execution-coordination) contract. The protocol integrates [verifiable delay function](#verifiable-delay-function) to combat front-running and order collision, allowing users to submit orders safely without worrying about a long and probabilistic order execution,

The exchange protocol implements 0x order schema on Ethereum and can be easily integrated with Ethereum, Cosmos, and Binance Chain DeFi ecosystems. However, unlike a standard 0x open orderbook model, users do not need to pay for Ethereum gas or exchange fees in ZRX. Injective also supports advanced exchange features such as [soft cancel](#soft-cancel-creation) and selective delay by default.

The protocol's [native token](#token-economics) will be used to maintain proof-of-stake security on the sidechain, reward order discovery and origination for nodes, and allow token holders to capture value on the success of the protocol via a token burn or distribution mechanism. The exchange protocol does not collect fees in native token by default but rather implements a negative spread model like most of the traditional centralized exchanges. The fees collected will undergo a periodic auction enforced on a smart contract to buy back the native token.


* describe problem
* fully-decentralized, open, permissionless
* front-running proof
* native token
* 0x trades, extensible to arbitrary DeFi
* gasless transactions

## Architecture
The protocol is comprised of three principal components: 1) the Injective sidechain relayer network, 2) Injective's trade execution coordinator contract (a smart contract on Ethereum), and (optionally) 3) a graphical front-end interface. Injective Protocol provides a fully decentralized sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator (TEC). Individuals can communicate orders/trades (e.g. through a front-end interface) to the sidechain relayer network which aggregates trades in a canonical manner which prevents front-running and then submits the on Injective's trade execution [coordinator contract](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#filter-contracts) which in turn executes and settles the trades through 0x.

<img alt="architecture.png" src="https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/.gitbook/assets/architecture.png?raw=true" width="700px"/>

Injective Protocol uses an application-specific sidechain relayer network to maintain a decentralized orderbook, front-running resistant trade execution coordinator, and order matching and execution engine. 

We refer to our exchange model as **sidechain order relay with on-chain settlement** - a decentralized implementation of the traditionally centralized [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture) used by nearly all central limit order book decentralized exchanges. 

## Decentralized Orderbook

The Injective sidechain hosts a decentralized, censorship-resistant orderbook which stores and relays orders. The application logic for the orderbook is in the [`orders`](https://github.com/InjectiveLabs/injective-core/tree/master/cosmos/x/orders) module which supports six distinct actions (i.e. state transitions in the form of [Msgs](https://godoc.org/github.com/cosmos/cosmos-sdk/types#Msg)): [make order creation](#make-order-creation), [take order creation](#take-order-creation), [soft cancel](#soft-cancel), [trading pair creation](#trading-pair-creation), [trading pair suspension](#trading-pair-suspension), and [trading pair resumption](#trading-pair-resumption). 

Each action results in a  [Tendermint procedure](#tendermint-procedure) (which we briefly include for completeness of understanding). Further documentation on the orders module and Tendermint can be found [here](https://github.com/InjectiveLabs/injective-core/blob/master/cosmos/x/orders/README.md) and [here](https://tendermint.com/docs/introduction/) respectively. 

#### Make Order Creation Steps
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

#### Take Order Creation Steps
1. A valid signed take order is created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer performs the same validation of the order as in the make order prodedure and also checks that:
	1. The make order fill amounts are fillable 
	2. The trade conforms to the negative spread model
	3. `VdfInput` of the order hash is valid (if supplied)
	4. `VdfOutput` is the valid output for the VDF applied on `VdfInput` for `VdfIterations` (if supplied)
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network (assuming step 3 passes)
5. The take order msg is handled and added to the pending queue of take orders to be submitted for that block

#### Soft Cancel Creation Steps
1. A valid signed cancel order is created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer validates the cancel order 
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network (assuming step 3 passes)
5. The make order in question is marked as soft-cancelled and take orders in the pending queue which include the soft-cancelled make order are removed. 

#### Trading Pair Creation Steps 
Injective will determine the proper governance and mechanism design for accepting trading pairs. <!---TODO!-->


Currently trading pairs can be created during the genesis transaction and using `MsgCreateTradePair`.

- requires filter contract sufficient balance
- trade pairs are stored by hash of the asset data
- asset data must match either ERC20, ERC721 or other supported asset types

#### Trading Pair Suspension Steps
TBD; determine suspension mechanism of trading pair through 1) governance procedure or 2) unplanned FC failure

Currently trade pair can be suspended by any validator using `MsgSuspendTradePair`

#### Trading Pair Resumption Steps
TBD; determine resumption mechanism of trading pair through 1) governance procedure

Currently trade pair can be resumed by any validator using `MsgResumeTradePair`

#### Tendermint Procedure
1. Each sidechain node calls [CheckTx](https://tendermint.com/docs/app-dev/abci-spec.html#checktx) on the transaction which performs [standard Tendermint checks](https://github.com/cosmos/cosmos-sdk/blob/master/docs/basics/tx-lifecycle.md#addition-to-mempool) on the transaction and adds/discards the transaction to the mempool.
2. Nodes reach consensus on on which transactions to include in each round through [Tendermint BFT](https://tendermint.com/docs/spec/consensus/consensus.html) and commit to a new block, fully executing the state transitions from each transaction.
   1. Each order message (transaction) is routed to its designated handler function which performs further checks and executes changes to the application store. 

### Relayer API
Each relayer can optionally support their own API allowing for 1) submission of orders to the sidechain and 2) query of data of the application state. The following implementation is provided out-of-the-box for relayers to use, but each relayer is free to provide their own API for their desired use case. 

Overall API documentation: [Injective Relayer API — Swagger](https://injective-tendermint-external-and-internal-api-2.api-docs.io/undefined/api/orderbyhash-relayer). The API surface is split into two namespaces:

Please note, that:

* `/api/v2` is a strict [Standard Relayer API v2](https://github.com/0xProject/standard-relayer-api/blob/master/http/v2.md) implementation for [Decentralized Orderbook](#decentralized-orderbook);
* `/api/rest` is a group of useful methods for Relayer management and custom scenarios, simply REST API.

### Trade Execution Coordination
**TODO**

#### Relayer Accounts and Eth-Tx Modules

A full validator node of the sidechain includes two important modules that are in charge of the propagation of matched orders onto the Ethereum state.

Relayer accounts module `x/accounts` is responsible for tracking online status of each active validators that are currently connected. This module prevents selecting an offline validator for order submission to Ethereum, when, for instance, validator has turned off its client gracefully or his node is in pending shutfown state.

To manage status and graceful "log-in"s and "log-off"s the module allows a validator node to periodically send a `MsgPing` transaction with current timestamp and version tag. It also allows the relayer to send a `MsgLogOff` transaction to gracefully set its status to offline. The accounts module provides methods for selecting a pool of currently active peers within a given timeframe and whose version tag is matching. This way, the application maintains a filtered pool of online peers (i.e. validators) that are ready and responsible for processing Ethereum transactions in the immediate future.

The Ethereum Tx module `x/ethtx` provides the main consensus engine for selecting Tx submitters and Tx reviewers. For each new Tendermint block (in the implemention of `EndBlocker`), a new submitter is chosen based on the pool of online validators returned by `x/accounts` at the current block height. Since all Tendermint nodes deterministically execute the same application logic on the same state, and the list of online peers is written in that state, there will always be consensus on 1) who's the submitter is at the current block and 2) who has been a submitter for each block in the past.

So, when a validator is ready to end a block, `x/ethtx` decides whether that validator is a submitter, or is a reviewer.

**1)** If selected as **submitter** (only one per block), the validator will query orders pending for submission using `x/orders`, encode and sign an Ethereum transaction that submits batched orders to the coodinator contract, deliver that transaction to a Geth instance or similar JSON-RPC API (Alchemy, Infura), and write the transaction hash into the sidechain state using `MsgSubmitTxHash`.

**2)** If selected as **reviewer** (the rest of peers), each selected validator will look back into history of N blocks, find a corresponding submitter for that block and get the tx hash. Then each reviewer queries the Ethereum state for this tx hash to check that the transaction was properly submitted and that all order hashes submitted match the list of pending orders for that past block of sidechain. Each chosen reviewer must submit a review to the sidechain using `MsgSubmitReview`. If verification has failed, the submitter is slashed (TBD).

### Validator Requirements (WIP)
To become a validator of the Injective sidechain, the participants have to accumulate more tokens than the minimum stake amount outlined in the protocol. The validator has the responsibility of proposing and verifying the sidechain block, facilitating the propagation, matching, and submission of orders. We utilize [Tendermint](https://cosmos.network/docs/cosmos-hub/validators/validator-faq.html#becoming-a-validator) consensus for the block production and validator selection process.
<!--https://docs.google.com/document/d/1PUnhciynPqXhyQqtywL5PeTz9mSDGWklp4-U7OajX-s/edit?pli=1#heading=h.ohv3ez9ge57k-->

#### Registration and Stake
The participants will submit a request `create-validator` on the sidechain to become validators. If the participants satisfy the requirements (most notably the `minimum self-delegation` amount or generally known as the minimum stake amount), they will be elidgible to become validators after a predefined number of blocks are mined. Once a participant becomes a validator, other participants in the sidechain can also delegate stakes to the validator's staking pool. Since our sidechain interacts with Ethereum, the validators and delegators alike will need to include their Ethereum address in their requests as well.

#### Validator Duties
Beyond participating in the consensus of the Injective sidechain, the validators also have the responsibility of verifying incoming orders, submitting orders to the Ethereum filter contract, and approving orders before it was submitted to 0x for settlement. When a trader submits an order with a VDF time proof to a validtor, the validator need to verify the proof and check for existing time proofs for the same order. If an existing order with shorter time proof is found, the validator can update the same order with the longer time proof. If no identical orders are found, the validator will verify and include the order in the upcoming block.

If the validator is the block proposer of a checkpoint block (one that aggregates orders since the last checkpoint block and submits to Ethereum <!--not sure how to phrase this-->), the block proposer has the responsibility of submitting the orders within the block to Ethereum. Although the block proposers need to provide the gas cost for the entire block, they can recoup the cost from staking reward and earnings from exchange fee.


#### Validator Rewards
Depending on the final implementation of the token economic model. The validator will either recieve both staking reward and exchange fee distribution in Injective's native token or solely the staking reward. Exchange fees are collected from the market pairs via negative spread model and are periodically auctioned off to market makers in exchange for Injective's native token. These tokens are either burned or distributed proportional by stake to the validators' staking pools depending on the final implementation.

#### Slashing
If validators fail to fulfill their responsibilities or perform maclicious activities, their entire staking pool is forfeited or slashed. Beyond Tendermint's standard slashing condition, Injective will also slash a block proposer's stake if he/she fails to submit the checkpoint block to Ethereum. This can be detected when other validators have yet to observe a confirmed Ethereum transaction containing the checkpoint block in question after a threshold of Ethereum blocks has been mined.

## Coordinator Contract
Trades are submitted by the sidechain relay network to Injective's trade execution coordinator smart contract which then executes and settles the trades on 0x. 

Injective's coordinator has 2 features that differentiate it from a traditional coordinator. 
1. The coodinator is decentralized and the submitting coordinator for each block is randomly chosen by the sidechain application logic. 
2. The coodinator submits the coordinated trades on behalf of takers, allowing traders to experience gasless trades. 

The flow for filling an order with our coordinator model is as follows:

1. Takers submit valid take orders which result in a trade with negative spread as described in [Take Order Creation](#take-order-creation). 
2. At a given block, the chosen coordinator for that block submits the pending queue of trades to the trade execution coordinator smart contract. 
3. The coordinator contract verifies negative spread on each trade and executes the trades on 0x. 
4. The coordinator contract returns [`FillResults[]`](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#fillresults). 

<img alt="trade-flow.png" src="https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/.gitbook/assets/trade-flow.png?raw=true" width="700px"/>

### Negative Spread 
A take order along with its corresponding make order(s) (i.e. a "trade") must result in bilateral negative spread with spread parameter `p = 1.002` in order to be accepted by the sidechain and coordinator contract. 

For two orders `OrderA` and `OrderB`, there exists negative spread if and only if the cost per unit bought (`OrderA.makerAssetAmount/OrderA.takerAssetAmount`) is greater than the profit per unit sold of the matched order (`OrderB.takerAssetAmount / (OrderB.makerAssetAmount * p^2)`), i.e. the following inequality must hold:

`p^2 * OrderA.makerAssetAmount * OrderB.makerAssetAmount >= OrderA.takerAssetAmount * OrderB.takerAssetAmount`

In our model, due to rounding reasons and limitations in decimal representations in Solidity, we represent `p^2 = 1.002^2` as `1.004`.

## Interface
Relayers can earn greater rewards by serving hosting an interface that is configured with their relayer API. Injective has provided two open-source front-end interface implementations allowing for users to interact with the protocol through a friendly graphical user interface. 

Our first implementation is a fork of the [0x-launch-kit-frontend](https://github.com/0xProject/0x-launch-kit-frontend) adapted to our protocol: 
* **[Injective launch kit implementation](https://github.com/InjectiveLabs/0x-frontend)**

Our second implementation is a more powerful and comprehensive interface catered towards the general public as well as more advanced users:
* **[Injective client implementation](https://github.com/InjectiveLabs/injective-client)**

## Governance
Governance occurs on two separate portions of our protocol: the sidechain application and the coordinator smart contract. 

### Sidechain Governance
The sidechain governance is built on top of the core Tendermint consensus. Validators for the Tendermint consensus process are incentivized by block reward and punished by slashing if a malicious behaviors were detected. This process is elaborated in [Validator Requirements](#validator-requirements). 

### Coordinator Contract Governance

<!-- ***structurally, would we enforce and require the same validators to be the pegzone/fisherman for the FC?*** -->
<!-- ***should validators the only ones calling for vote? or anyone can call for votes*** -->
The coordinator contract only approves blocks of trade that has accumulated enough signature from a supermajority of validators from the sidechain. The coordinator contract maintains a list of validators that can be updated from the sidechain if there are supermajority agreement from the previous list of validators. 

Once the trades are approved, the coordinator contract can submit <!-- ***not sure if this is accurate***  -->the trades to 0x for settlement.
#### Voting mechanism
Injective's native token holder can participate in governing the coordinator contract on Ethereum. They have the power to vote on key decisions such as protocol upgrade, listing, fee schedule, and modifying other key variables inThe protocol's [native token](#token-economics) will be used to maintain proof-of-stake security on the sidechain, reward order discovery and origination for nodes, and allow token holders to capture value on the success of the protocol via a token burn or distribution mechanism. The exchange protocol does not collect fees in native token by default but rather implements a negative spread model like most of the traditional centralized exchanges. The fees collected will undergo a periodic auction enforced on a smart contract to buy back the native token.
 the exchange. Our protocol allows token holders to create proposals that can be voted on by the community with their tokens. 
#### Creating a Proposal
In order to create a proposal for all token holders to vote on, a proposer must lockup more than the `Minimum_proposal_requirement` to open a voting period.

After a proposal is successfully created, a pending period begins where the proposal must accumulate enough lockup to surpass the `Minimum_referendum_requirement` before `Proposal_pending_period` (denominated in blocks) expires. The `Minimum_referendum_requirement` will be calculated as a percentage of the circulating supply of Injective's native token. If the proposal surpasses the requirement before the pending period ends, it will successfully become a referendum that the general token holders can vote on.

Once then pending period ends, the lockup can be withdrew regardless of the outcome. Depending on the proposal's subject matter, the variables `Minimum_referendum_requirement` and `Proposal_pending_period` may be different.
#### Voting on a Proposal
Once a proposal becomes a referendum, a `Referendum_period` will begin. During the period, token holders can vote by creating a `Vote` transaction. By doing so, the token holder is signaling  yay or nay on the proposal with the voting power proportional to their wallet balance. Once a vote is finalized, anyone can call the smart contract to tally the vote and reach a decision. However, if the final yay vote does not pass the `Minimum_yes_requirement`, the referendum is considered fail due to the lack of participation. Like `Minimum_referendum_requirement`, the `Minimum_yes_requirement` is based on a percentage of the token's circulating supply.

If a proposal is successfully voted into implementation, then the smart contract will enforce the decision deterministically.

## Token Economics
The full specifications for Injective Protocol's token economics can be found [here](https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/token-economics.md). 

Injective Protocol's native token is used for the following purposes:

1. Proof-of-stake reward
Validators can stake with the token and receive block reward proportional to stake.
2. Fee Distribution
Exchange fees collected via negative spread model are periodically auctioned off through smart contract to buy back Injective's native token for reward distribution. The tokens are first distributed to relayers proportional by orders discovered/originated, and then to validators proportional by stake. Part of the tokens may also be burned instead of distributing to validators depending on final implementation. 


We will implement these features in future iterations.

3. Market Maker Incentives
Make orders will receive a net positive fee rebate to incentivize liquidity.
4. Fee Discount
Token holders may receive discount on exchange fees.
5. Governance
Reference [Coordinator Contract Governance](#coordinator-contract-governance)


## Miscellaneous

### Front-Running Prevention

### Verifiable Delay Function 
#### Implementation

In our protocol,``t`` is the time parameter for proof of elapsed time. In the context of Sloth, ``t`` 
is the number of modular square root evaluation. ``t`` can be computed incrementally for it to become a time proof.


After submitting the take order information adhering to 0x schema, the user will generate `x` value by creating the `orderHash` from `Order`.

Using `x`, user begins to calculate VDF (Sloth for example): ``sqrt(x)-> y , sqrt(y)-> y_1 , sqrt(y_1)->y_2 ...`` until it reaches a satisfactory checkpoint `y_t`. User then submits `y_t, t , x , order_Hash` to a relayer so that it can map them to the order information the user submitted previously. The relayer will then perform `vdf_verify`.
to ensure the validity of the user's VDF proof. In sloth's case, the relayer can simply calculate `((y_t)^2)^(t+1)==x`. 

After `Vdf_verify`, the relayer will append `t` to the user's existing order information. Since traders are allowed to submit multiple transactions for time proofs (ex: `tx_1: {y_t , t ,x} tx_2: {y_2t , 2t , x} tx_3: {y_3t , 3t , x}`), the relayer should also check if the `t` is bigger than the existing `t` attached to the trader's information before performing any VDF verification.

Before a block is mined, the relayers match take orders with make orders on a `t`-priority basis. Meaning that for each make order, the taker orders are sorted by descending `t` and filled in that order.


#### Interaction

Our client will be interacting with the VDF candidates under through `vdf_interface.go`. The exported functions are:
```
Sloth_fixed_delay(p_parameter string, starting_value string, iteration string) string 
Sloth_eval(p_parameter string, starting_value string, iteration string) string  
Sloth_verify(p_parameter string, starting_value string, iteration string, ending_value string) bool 
```
We created `Sloth_fixed_delay` for testing purposes. 

`Sloth_eval` takes in:

prime number: `p_parameter : type string`
`x` or starting value: `starting_value: type string`
`t: type string` iteration count: `iteration: type string`

and outputs

`y_t` or ending value at iteration `t`: `ending_value: type string`

`Sloth_verify` takes in:

prime number: `p_parameter : type string`
`x` or starting value: `starting_value: type string`
`t: type string` iteration count: `iteration: type string`
`y_t` or ending value at iteration `t`: `ending_value: type string`

and outputs

the result of the verification:  `resulte : type bool`

Within this file, we have stored a few prime numbers for demonstration purposes, these numbers will not be in production.

`vdf_interface` uses the candidates in `\candidates` subdirectory. Currently we are only using `sloth.go`. In `sloth.go`, the relevant functions exported are:
```
Eval( args[3]string ) string
Verify( args[4]string ) bool
```

The `string` arguments for `Eval` are `[ p , x , t ]` and the function returns `y_t` as the output. This function will evaluate the VDF using the Sloth candidate with the ending value `y_t` as the output. 

`Verify` arguments are ` [ p , x , t , y ] ` and the function returns a boolean as the output. This function takes all the VDF parameters and returns whether the ending value `y_t` was correct.


