# Injective Protocol Architecture

Injective Protocol is comprised of four principal components: 

1\) Injective Chain

2\) Injective's smart contracts on Ethereum

3) Injective API nodes

4\) Front-end interface

![](../.gitbook/assets/architecture.png)

## Injective Chain

The Injective Chain is an fully-decentralized sidechain relayer network which serves as a layer-2 derivatives platform, trade execution coordinator (TEC), and decentralized orderbook. The core consensus is Tendermint-based. 

### Layer-2 Derivatives Platform 

The Injective Chain supports building generalized derivatives/DeFi applications through two avenues: the Injective Futures Protocol and general smart contracts. 

**Injective Futures Protocol**

The Injective Futures Protocol is deployed on the Injective Chain as a Cosmos-SDK based application. This protocol enables traders to create, enter into, and execute decentralized perpetual swap contracts and CFDs on any arbitrary market. 

**Smart Contracts**

The Injective Chain provides a two-way Ethereum peg-zone for Ether and ERC-20 tokens to be transferred to the Injective Chain as well as an EVM-compatible execution environment for DeFi applications. The peg-zone is based off [Peggy](https://github.com/cosmos/peggy) and the EVM execution is based off [https://github.com/chainsafe/ethermint](https://github.com/chainsafe/ethermint). 

### Trade Execution Coordinator



### Decentralized Orderbook



Injective Protocol provides a fully decentralized sidechain relayer network which serves as a decentralized orderbook and trade execution coordinator \(TEC\). 

 (trade execution coordinator, bridge contracts)

Injective Protocol uses an application-specific sidechain relayer network to maintain a decentralized orderbook and front-running resistant trade execution coordinator. 

We refer to our exchange model as **sidechain order relay with on-chain settlement** - a decentralized implementation of the traditionally centralized [off-chain order relay](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#architecture) used by nearly all central limit order book decentralized exchanges. 

The Injective sidechain hosts a decentralized, censorship-resistant orderbook which stores and relays orders and also serves as a decentralized trade execution coordinator (TEC). 

Each node of the Injective chain runs RelayerDaemon which maintains a decentralized orderbook and trade execution coordinator. Users can interact with the node directly through Tendermint transactions or through the [RelayerAPI](https://api.injective.dev/#operation/Relayer#getAccount) which relayers can optionally support. 

## Decentralized Orderbook

The application logic for the orderbook is in the [`orders`](https://github.com/InjectiveLabs/injective-core/tree/master/cosmos/x/orders) module which supports six distinct actions \(i.e. state transitions in the form of [Msgs](https://godoc.org/github.com/cosmos/cosmos-sdk/types#Msg)\): [make order creation](sidechain.md#make-order-creation) and [soft cancel](sidechain.md#soft-cancel).

Each action results in a [Tendermint procedure](sidechain.md#tendermint-procedure) \(which we briefly include for completeness of understanding\). Further documentation on the orders module and Tendermint can be found [here](https://github.com/InjectiveLabs/injective-core/blob/master/cosmos/x/orders/README.md) and [here](https://tendermint.com/docs/introduction/) respectively.

### Make Order Creation Steps

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
   3. The maker's  `makerAssetData`  approval amount should be greater than or equal to the `makerAssetAmount` of the order
   4. The signature validity for the hash of the order
   5. The trading pair is supported by the sidechain
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network \(assuming step 3 passes\)
5. The make order msg is handled and added to the orderbook. 

### Soft Cancel Creation Steps

1. A valid signed cancel order is created
2. The order is submitted to the sidechain through a HTTP POST call to a relayer's endpoint which then forwards the order to the network
3. The relayer validates the cancel order 
4. The relayer broadcasts a [Tendermint/Cosmos SDK Tx](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L38) containing the order to its peers in the sidechain network \(assuming step 3 passes\)
5. The make order in question is marked as soft-cancelled and take orders in the pending queue which include the soft-cancelled make order are removed. 



## Decentralized Trade Execution Coordinator 

The Injective TEC is a decentralized implementation of the [0x 3.0 Coordinator](https://github.com/0xProject/0x-protocol-specification/blob/master/v3/coordinator-specification.md) model. 

To place a trade, one must send a signed 0x transaction to the sidechain which will then either approve or reject the transaction. If the transaction is approved, the trader can then submit their trade to Ethereum for execution and settlement. 