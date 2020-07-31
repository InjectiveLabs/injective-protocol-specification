# Make Orders

In the Injective Perpetuals Protocol, there are two main types of orders: maker orders and taker orders. **Make orders** are stored on Injective's decentralized orderbook on the Injective Chain while **Take orders** are immediately executed against make orders on the Injective Perpetuals Contract.

## **Order Message Format**

The Injective Perpetuals Protocol leverages the [0x Order Message format](https://github.com/0xProject/0x-protocol-specification/blob/master/v3/v3-specification.md#order-message-format) for the external interface to represent a make order for a derivative position, i.e. a cryptographically signed message expressing an agreement to enter into a derivative position under specified parameters.

A make order message consists of the following parameters:

| Parameter | Type | Description |
| :--- | :--- | :--- |
| makerAddress | address | Address that created the order. |
| takerAddress | address | Empty. |
| feeRecipientAddress | address | Address of the recipient of the order transaction fee. |
| senderAddress | address | Empty. |
| makerAssetAmount | uint256 | The contract price \($$P_{contract}$$\), i.e. the price of 1 contract denominated in base currency. |
| takerAssetAmount | uint256 | The $$quantity$$ of contracts the maker seeks to obtain. |
| makerFee | uint256 | The amount of $$margin$$ denoted in base currency the maker would like to post/risk for the order. |
| takerFee | uint256 | Empty. |
| expirationTimeSeconds | uint256 | Timestamp in seconds at which order expires. |
| salt | uint256 | Arbitrary number to facilitate uniqueness of the order's hash. |
| makerAssetData | bytes | The first 32 bytes contain the `marketID` of the market for the position if the order is LONG, empty otherwise.  Right padded with 0's to be 36 bytes |
| takerAssetData | bytes | The first 32 bytes contain the `marketID` of the market for the position if the order is LONG, empty otherwise.  Right padded with 0's to be 36 bytes |
| makerFeeAssetData | bytes | Empty. |
| takerFeeAssetData | bytes | Empty. |

In a given perpetual market specified by `marketID`, an order encodes the willingness to purchase `quantity` contracts in a given direction \(long or short\) at a specified contract price $$P_{contract}$$ using a specified amount of `margin` of base currency as collateral.

## Order Validation

### getOrderRelevantState

```javascript
/// @dev Fetches all order-relevant information needed to validate if the supplied order is fillable.
/// @param order The order structure
/// @param signature Signature provided by maker that proves the order's authenticity.
/// @param indexPrice The index price to use as a reference. If 0, use the market's existing index price.
/// @return The orderInfo (hash, status, and `takerAssetAmount` already filled for the given order),
/// fillableTakerAssetAmount (amount of the order's `takerAssetAmount` that is fillable given all on-chain state),
/// and isValidSignature (validity of the provided signature).
function getOrderRelevantState(
    LibOrder.Order memory order,
    bytes memory signature,
    uint256 indexPrice
)
    public
    view
    returns (
        LibOrder.OrderInfo memory orderInfo,
        uint256 fillableTakerAssetAmount,
        bool isValidSignature
    )
{
```

Logic:

Calling `getOrderRelevantState` will perform the following steps:

1. Set the hash of the `order`  in `orderInfo.orderHash`
2. Set the quantity of contracts of the `order` that have been filled in `orderInfo.orderTakerAssetFilledAmount`
3. Set the order status of the order in `orderInfo.orderStatus` according to the following conditions:
   1. If the order has been cancelled, the `orderStatus` will be `CANCELLED`.
   2. If the order has been fully filled (i.e. `orderInfo.orderTakerAssetFilledAmount` equals `order.takerAssetAmount`, the `orderStatus` will be `FULLY_FILLED`.
   3. If the order has expired, the `orderStatus` will be `EXPIRED`.
   4. If the order's margin (`order.makerFee`) does not satisfy the [initial margin requirement](./keyterms.md#initial-margin-requirement), the `orderStatus` will be `INVALID_MAKER_ASSET_AMOUNT`. Note: the index price used in this calculation is the inputted `indexPrice`.
   5. Otherwise if the order is fillable, the `orderStatus` will be `FILLABLE` and the `fillableTakerAssetAmount` to equal the quantity of contracts that remain fillable (i.e. `order.takerAssetAmount - orderInfo.orderTakerAssetFilledAmount`).
4. Check whether or not the signature is valid and set `isValidSignature` to true if valid. Otherwise, set to false and set the `orderStatus` to `INVALID`.
5. If the `orderStatus` from the previous steps is `FILLABLE`, check that the order maker has sufficient balance of baseCurrency in his freeDeposits (his `availableMargin`) to fill `fillableTakerAssetAmount` contracts of the order.
   * If the maker does not have at least `order.makerFee * fillableTakerAssetAmount / order.takerAssetAmount + fillableTakerAssetAmount * order.makerAssetAmount * makerTxFee` amount of balance to the fill the order, `fillableTakerAssetAmount` is set to the maximum quantity of contracts fillable which equals `availableMargin * order.TakerAssetAmount / order.MakerFee + order.TakerAssetAmount * makerTxFee`. If this value is zero, the `orderStatus` will be `INVALID_TAKER_ASSET_AMOUNT`.

### getOrderRelevantStates

Sequentially calls `getOrderRelevantState`.


```js
/// @dev Fetches all order-relevant information needed to validate if the supplied orders are fillable.
/// @param orders Array of order structures
/// @param signatures Array of signatures provided by makers that prove the authenticity of the orders.
/// @return The ordersInfo (array of the hash, status, and `takerAssetAmount` already filled for each order),
/// fillableTakerAssetAmounts (array of amounts for each order's `takerAssetAmount` that is fillable given all on-chain state),
/// and isValidSignature (array containing the validity of each provided signature).
/// NOTE: Expects each of the orders to be of the same marketID, otherwise may return incorrect information
function getOrderRelevantStates(LibOrder.Order[] memory orders, bytes[] memory signatures)
  public
  view
  returns (
    LibOrder.OrderInfo[] memory ordersInfo,
    uint256[] memory fillableTakerAssetAmounts,
    bool[] memory isValidSignature
    );
```

### getMakerOrderRelevantStates

TODO: document

```javascript
/// @dev Fetches all order-relevant information needed to validate if the supplied orders are fillable.
/// @param orders Array of order structures
/// @param signatures Array of signatures provided by makers that prove the authenticity of the orders.
/// @param makerAddress Address of maker to check.
/// @return The ordersInfo (array of the hash, status, and `takerAssetAmount` already filled for each order),
/// fillableTakerAssetAmounts (array of amounts for each order's `takerAssetAmount` that is fillable given all on-chain state),
/// isValidSignature (array containing the validity of each provided signature), and availableMargin (amount of available
/// base currency usable as margin after margin needs of the `orders` are satisfied).
/// NOTE: Expects each of the orders to be of the same marketID, otherwise may return incorrect information
function getMakerOrderRelevantStates(
    LibOrder.Order[] memory orders,
    bytes[] memory signatures,
    address makerAddress
)
    public
    view
    returns (
        LibOrder.OrderInfo[] memory ordersInfo,
        uint256[] memory fillableTakerAssetAmounts,
        bool[] memory isValidSignature,
        uint256 availableMargin
    )
```

### OrderInfo

```javascript
struct OrderInfo {
    uint8 orderStatus;                    // Status that describes order's validity and fillability.
    bytes32 orderHash;                    // EIP712 hash of the order (see LibOrder.getOrderHash).
    uint256 orderTakerAssetFilledAmount;  // Amount of order that has already been filled.
}
```

### Order Status

```javascript
enum OrderStatus {
    INVALID,
    INVALID_MAKER_ASSET_AMOUNT,
    INVALID_TAKER_ASSET_AMOUNT,
    FILLABLE,
    EXPIRED,
    FULLY_FILLED,
    CANCELLED
}
```
