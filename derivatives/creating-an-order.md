In the Injective Perpetuals Protocol, there are two main types of orders: maker orders and taker orders. **Make orders** are stored on Injective's decentralized orderbook on the Injective Chain while **Take orders** are immediately executed against make orders on the Injective Perpetuals Contract.
# **Make Order Message Format**

The Injective Perpetuals Protocol leverages the [0x Order Message format](https://github.com/0xProject/0x-protocol-specification/blob/master/v3/v3-specification.md#order-message-format) for the external interface to represent a make order for a derivative position, i.e. a cryptographically signed message expressing an agreement to enter into a derivative position under specified parameters. 

A make order message consists of the following parameters:

| Parameter                           | Type    | Description                                                  |
| ----------------------------------- | ------- | ------------------------------------------------------------ |
| makerAddress                        | address | Address that created the order.                              |
| takerAddress                        | address | Empty. |
| feeRecipientAddress                 | address | Address of the recipient of the order transaction fee.        |
| senderAddress     | address | Empty. |
| makerAssetAmount                    | uint256 | The contract price ($$P_{contract}$$), i.e. the price of 1 contract denominated in base currency. |
| takerAssetAmount                    | uint256 | The $$quantity$$ of contracts the maker seeks to obtain. |
| makerFee                            | uint256 | The amount of $$margin$$ denoted in base currency the maker would like to post/risk for the order. |
| takerFee                            | uint256 | Empty. |
| expirationTimeSeconds               | uint256 | Timestamp in seconds at which order expires.                 |
| salt                       | uint256 | Arbitrary number to facilitate uniqueness of the order's hash. |
| makerAssetData        | bytes   | The first 32 bytes contain the `marketID` of the market for the position if the order is LONG, empty otherwise.  Right padded with 0's to be 36 bytes |
| takerAssetData        | bytes   | The first 32 bytes contain the `marketID` of the market for the position if the order is LONG, empty otherwise.  Right padded with 0's to be 36 bytes |
| makerFeeAssetData | bytes   | Empty. |
| takerFeeAssetData | bytes   | Empty. |


In a given perpetual market specified by `marketID`, an order encodes the willingness to purchase `quantity` contracts in a given direction (long or short) at a specified contract price $$P_{contract}$$ using a specified amount of `margin` of base currency as collateral. 




