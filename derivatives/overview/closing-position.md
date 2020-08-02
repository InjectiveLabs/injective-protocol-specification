# Closing Position

## closePosition

```javascript
/// @dev Closes the input position.
/// @param positionID The positionID of the position being closed
/// @param orders The orders to use to settle the position
/// @param quantity The quantity of contracts being used in the order
/// @param signatures The signatures of the orders signed by makers.
function closePosition(
    uint256 positionID,
    LibOrder.Order[] memory orders,
    uint256 quantity,
    bytes[] memory signatures
) external (PositionResults[] memory pResults, CloseResults memory cResults);
```

**Logic**

Calling `closePosition` will perform the following steps:

1. Query the oracle to obtain the most recent price and funding fee.
2. Execute funding payments on the existing position and then update the existing position state.
3. Check that the existing `position` \(referenced by `positionID`\) is valid and can be closed. 
4. Create the Makers' Positions. For each order `i`:
   1. If the order has been used previously, execute funding payments on the existing position and then update the existing position state. Otherwise, create a new account with a corresponding new position with the `pResults[i].quantity` contracts for the maker and log a `FuturesPosition` event.
   2. Transfer `pResults[i].marginUsed + pResults[i].fee` of base currency from the maker to the contract to create \(or add to\) the maker's position.
   3. Allocate `pResults.marginUsed` for the new position margin and allocate `relayerFeePercentage` of the `pResults.fee` to the fee recipient \(if specified\) and the remaining `pResults.fee` to the insurance pool.
5. Close the existing position. 
   1. Calculate the PNL per contract \(`contractPNL`\) which equals `averageClosingPrice - position.contractPrice` for longs and `position.contractPrice - averageClosingPrice` for shorts, where:
      * `averageClosingPrice = (orders[i].contractPrice * results[0].quantity + orders[n-1].contractPrice * results[n-1].quantity)/(cResults.quantity)` 
      * `cResults.quantity = results[0].quantity + ... + results[n-1].quantity` . Note that `cResults.quantity <= quantity`. 
   2. Transfer `cResults.payout` to the owner of the `position` and update the position state. 
      * `cResults.payout = cResults.quantity * (position.margin / position.quantity + contractPNL)` 
   3. Log a `FuturesClose` event. 

## closePositionOrKill

```javascript
/// @dev Closes the input position and revert if the entire `quantity` of contracts cannot be closed.
/// @param positionID The positionID of the position being closed
/// @param orders The orders to use to settle the position
/// @param quantity The quantity of contracts being used in the order
/// @param signatures The signatures of the orders signed by makers.
function closePositionOrKill(
  uint256 positionID,
  LibOrder.Order[] memory orders,
  uint256 quantity,
  bytes[] memory signatures
) external returns (PositionResults[] memory pResults, CloseResults memory cResults);
```

**Logic**

Calling `closePositionOrKill` will perform the same steps as `closePosition` but will revert if the entire `quantity` of contracts inputted cannot be closed.

