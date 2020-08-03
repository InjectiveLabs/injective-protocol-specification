# Liquidation



## liquidatePositionWithOrders 

```jsx
/// @dev Closes the input position.
/// @param positionID The ID of the position to liquidate.
/// @param quantity The quantity of contracts of the position to liquidate.
/// @param orders The orders to use to liquidate the position.
/// @param signatures The signatures of the orders signed by makers.
function liquidatePositionWithOrders(
  uint256 positionID,
  uint256 quantity,
  LibOrder.Order[] memory orders,
  bytes[] memory signatures
) external returns (PositionResults[] memory pResults, LiquidateResults memory lResults){
```

Logic

Calling `liquidatePositionWithOrders` will perform the following steps:

1. Query the oracle to obtain the most recent price and funding fee.
2. Execute funding payments on the existing position and then update the existing position state.
3. Check that the existing `position` (referenced by `positionID`) is valid and can be liquidated (i.e. that the [maintenance margin requirement](keyterms.md#maintenance-margin-requirement) is breached. 
4. Create the Makers' Positions. 
	1. For each order `i`:
		1. If the order has been used previously, execute funding payments on the existing position and then update the existing position state. Otherwise, create a new account with a corresponding new position with the `pResults[i].quantity` contracts for the maker and log a `FuturesPosition` event.`
		2. Transfer `pResults[i].marginUsed + pResults[i].fee` of base currency from the maker to the contract to create (or add to) the maker's position.
		3. Allocate `pResults.marginUsed` for the new position margin and allocate `relayerFeePercentage` of the `pResults.fee` to the fee recipient (if specified) and the remaining `pResults.fee` to the insurance pool.
	2. `lResults.quantity` equals the total quantity of contracts created across the orders from the previous step, i.e. `lResults.quantity = pResults[0].quantity + ... + pResults[n-1].quantity` . Note that `lResults.quantity <= quantity`. 
	3.  `lResults.liquidationPrice` equals the weighted average price, i.e. `lResults.liquidationPrice = (orders[i].contractPrice * pResults[0].quantity + orders[n-1].contractPrice * pResults[n-1].quantity)/(lResults.quantity)` 
		* To liquidate a long position, `lResults.liquidationPrice` must be greater than or equal to the index price. 
		* To liquidate a short position, `lResults.liquidationPrice` must be less than or equal to the index price. 

5. Close the `lResults.quantity` quantity contracts of the existing position. 
	1. Calculate the trader's loss (`position.margin / position.quantity * quantity`) and update the state of the trader's position. 
	2. Calculate the PNL per contract (`contractPNL`) which equals `lResults.liquidationPrice - position.contractPrice` for longs and ` position.contractPrice - averageClosingPrice ` for shorts.
	3. Calculate the total payout from the position`cResults.payout = cResults.quantity * (position.margin / position.quantity + contractPNL) `
		1. Allocate half of the payout to the liquidator and half to the insurance fund
	4. Decrement the position's remaining margin by `position.margin / position.quantity * (position.quantity - quantity)`
	5. Emit a `FuturesLiquidation` event. 

