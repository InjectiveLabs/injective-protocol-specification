# Filling Orders

Orders can be filled by calling the following methods on the `InjectiveFutures` contract

### fillOrder

This is the most basic way to fill an order. All of the other methods call `fillOrder` under the hood with additional logic. This function will attempt to execute `quantity` contracts of the `order` specified by the caller. However, if the remaining fillable amount is less than the `quantity` specified, the remaining amount will be filled. Partial fills are allowed when filling orders.

```javascript
/// @dev Fills the input order.
/// @param order The make order to be executed.
/// @param quantity Desired quantity of contracts to execute.
/// @param margin Desired amount of margin (denoted in baseCurrency) to use to fill the order.
/// @param signature The signature of the order signed by maker.
/// @return accountID The accountID of the newly created account, if there is one.
function fillOrder(
    LibOrder.Order memory order,
    uint256 quantity,
    uint256 margin,
    bytes memory signature
) public returns (bytes32);
```

**Logic**

Calling `fillOrder` will perform the following steps:

1. Query the oracle to obtain the most recent price and funding fee.
2. Query the state and status of the order

2. Revert if the order is unfillable (invalid context, expired, cancelled, fully filled, invalid signature)

[Initial Margin Requirement](./keyterms.md#initial-margin-requirement)



This will immediately enter the workflow of establishing two positions - one for the maker and one for the taker. 

2. By doing so, the futures contract transfers `margin` amount of the base currency specified by the make order and take order using ERC-20 the `transferFrom` call and deposits the tokens into their balance. 

3. The positions are attempted to be created for both parties. If the following conditions succeed, then the positions are created. If any one of these checks fail the entire transaction is reverted.



## **Market Orders**

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