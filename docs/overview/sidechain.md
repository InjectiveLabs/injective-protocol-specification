## Sidechain
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

Overall API documentation: [Injective Relayer API ](https://injective-tendermint-external-and-internal-api-2.api-docs.io/undefined/api). The API surface is split into two namespaces:

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