# Take Orders

Orders can be filled by calling the following methods on the `InjectiveFutures` contract

## fillOrder

This is the most basic way to fill an order. All of the other methods call `fillOrder` under the hood with additional logic. This function will attempt to execute `quantity` contracts of the `order` specified by the caller. However, if the remaining fillable amount is less than the `quantity` specified, the remaining amount will be filled. Partial fills are allowed when filling orders.

```javascript
/// @dev Fills the input order.
/// @param order The make order to be executed.
/// @param quantity Desired quantity of contracts to execute.
/// @param margin Desired amount of margin (denoted in baseCurrency) to use to fill the order.
/// @param signature The signature of the order signed by maker.
/// @return fillResults
function fillOrder(
    LibOrder.Order memory order,
    uint256 quantity,
    uint256 margin,
    bytes memory signature
) external returns (FillResults memory);
```

**Logic**

Calling `fillOrder` will perform the following steps:

1. Query the oracle to obtain the most recent price and funding fee.
2. Query the state and status of the order with [`getOrderRelevantState`](keyterms/#getorderrelevantstate).
3. Revert if the orderStatus is not `FILLABLE`. 
4. Create the Maker's Position.
   1. If the order has been used previously, execute funding payments on the existing position and then update the existing position state. Otherwise, create a new account with a corresponding new position for the maker and log a `FuturesPosition` event.
   2. Transfer `fillResults.makerMarginUsed + fillResults.feePaid` of base currency from the maker to the contract to create \(or add to\) the maker's position.
   3. Allocate `fillResults.makerMarginUsed` for the new position margin and allocate `relayerFeePercentage` of the `fillResults.makerFeePaid` to the fee recipient \(if specified\) and the remaining `fillResults.feePaid` to the insurance pool.
5. Create the Taker's position.
   1. Create a new account with a corresponding new position for the taker and log a `FuturesPosition` event.
   2. Transfer `margin + fillResults.takerFeePaid` of base currency from the taker to the contract to create the taker's position.  
   3. Allocate  `margin`  for the new position margin and allocate `relayerFeePercentage` of the `fillResults.takerFeePaid` to the fee recipient \(if specified\) and the remaining `fillResults.takerFeePaid` to the insurance pool.

## fillOrKillOrder

`fillOrKillOrder` can be used to fill an order while guaranteeing that the specified amount will either be filled or the call will revert.

```javascript
/// @dev Fills the input order. Reverts if exact quantity not filled
/// @param order The make order to be executed.
/// @param quantity Desired quantity of contracts to execute.
/// @param margin Desired amount of margin (denoted in baseCurrency) to use to fill the order.
/// @param signature The signature of the order signed by maker.
/// return results The fillResults
function fillOrKillOrder(
  LibOrder.Order memory order,
  uint256 quantity,
  uint256 margin,
  bytes memory signature
) external returns (FillResults memory);
```

**Logic**

Calling `fillOrKillOrder` will perform the following steps:

1. Call `fillOrder` with the passed in inputs
2. Revert if `fillResults.quantityFilled` does not equal the passed in `quantity`

## batchFillOrders

`batchFillOrders` can be used to fill multiple orders in a single transaction.

```text
/// @dev Executes multiple calls of fillOrder.
```

**Logic**

Calling `batchFillOrders` will perform the following steps:

1. Sequentially call `fillOrder` for each element of `orders`, passing in the order, fill amount, and signature at the same index.

## batchFillOrKillOrders

`batchFillOrKillOrders`can be used to fill multiple orders in a single transaction while guaranteeing that the specified amounts will either be filled or the call will revert.

```javascript
/// @dev Executes multiple calls of fillOrKill orders.
/// @param orders The make order to be executed.
/// @param quantities Desired quantity of contracts to execute.
/// @param margins Desired amount of margin (denoted in baseCurrency) to use to fill the order.
/// @param signatures The signature of the order signed by maker.
/// return results The fillResults
function batchFillOrKillOrders(
  LibOrder.Order[] memory orders,
  uint256[] memory quantities,
  uint256[] memory margins,
  bytes[] memory signatures
) external returns (FillResults[] memory results)
```

**Logic**

Calling `batchFillOrKillOrders` will perform the following steps:

## batchFillOrdersSinglePosition

`batchFillOrdersSinglePosition` can be used to

```javascript
/// @dev Executes multiple calls of fillOrder but creates only one position for the taker.
/// @param orders The make order to be executed.
/// @param quantities Desired quantity of contracts to execute.
/// @param margins Desired amount of margin (denoted in baseCurrency) to use to fill the order.
/// @param signatures The signature of the order signed by maker.
/// return results The fillResults
function batchFillOrdersSinglePosition(
    LibOrder.Order[] memory orders,
  uint256[] memory quantities,
  uint256[] memory margins,
  bytes[] memory signatures
) external returns (FillResults[] memory results)
```

**Logic**

Calling `batchFillOrdersSinglePosition` will perform the following steps:

## batchFillOrKillOrdersSinglePosition

`batchFillOrKillOrdersSinglePosition` can be used to

```javascript
/// @dev Executes batchFillOrKillOrders but creates only one position for the taker.
/// @param orders The make order to be executed.
/// @param quantities Desired quantity of contracts to execute.
/// @param margins Desired amount of margin (denoted in baseCurrency) to use to fill the order.
/// @param signatures The signature of the order signed by maker.
/// return results The fillResults
function batchFillOrKillOrdersSinglePosition(
  LibOrder.Order[] memory orders,
  uint256[] memory quantities,
  uint256[] memory margins,
  bytes[] memory signatures
) external returns (FillResults[] memory results)
```

**Logic**

Calling `batchFillOrKillOrdersSinglePosition` will perform the following steps:

## **marketOrders**

`marketOrders` can be used to

```javascript
/// @dev marketOrders executes the orders
/// @param orders Array of order specifications.
/// @param quantity Desired quantity of contracts to execute.
/// @param margin Desired amount of margin (denoted in baseCurrency) to use to fill the orders.
/// @param signatures Proofs that orders have been signed by makers.
/// @return accountID The accountID of the newly created account, if there is one.
function marketOrders(
    LibOrder.Order[] memory orders,
    uint256 quantity,
    uint256 margin,
    bytes[] memory signatures
) public returns (bytes32);
```

**Logic**

Calling `marketOrders` will perform the following steps:

## **marketOrdersOrKill**

`marketOrdersOrKill` can be used to

```javascript
/// @dev marketOrdersOrKill executes the orders and reverts if the `quantity` of contracts are not filled
/// @param orders Array of order specifications.
/// @param quantity Desired quantity of contracts to execute.
/// @param margin Desired amount of margin (denoted in baseCurrency) to use to fill the orders.
/// @param signatures Proofs that orders have been signed by makers.
function marketOrdersOrKill(
  LibOrder.Order[] memory orders,
  uint256 quantity,
  uint256 margin,
  bytes[] memory signatures
) public returns (FillResults[] memory results);
```

**Logic**

Calling `marketOrdersOrKill` will perform the following steps:

