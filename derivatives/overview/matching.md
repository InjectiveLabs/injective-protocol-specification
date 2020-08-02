# Matching Orders

Two orders of opposing directions can directly be matched if they have a negative spread.

## matchOrders

This is the most basic way to match two orders.

```javascript
/// @dev Matches the input orders.
/// @param leftOrder The order to be settled.
/// @param rightOrder The order to be settled.
/// @param leftSignature The signature of the order signed by maker.
/// @param rightSignature The signature of the order signed by maker.
function matchOrders(
  LibOrder.Order memory leftOrder,
  LibOrder.Order memory rightOrder,
  bytes memory leftSignature,
  bytes memory rightSignature
) external;
```

**Logic**

Calling `matchOrders` will perform the following steps:

## multiMatchOrders

`multiMatchOrders` can be used to

```javascript
/// @dev Matches the input orders and only creates one position for the `rightOrder` maker.
/// @param leftOrders The orders to be settled.
/// @param rightOrder The order to be settled.
/// @param leftSignatures The signatures of the order signed by maker.
/// @param rightSignature The signature of the order signed by maker.
function multiMatchOrders(
  LibOrder.Order[] memory leftOrders,
  LibOrder.Order memory rightOrder,
  bytes[] memory leftSignatures,
  bytes memory rightSignature
) external;
```

**Logic**

Calling `multiMatchOrders` will perform the following steps:

## batchMatchOrders

`batchMatchOrders` can be used to

```javascript
/// @dev Matches the input orders.
/// @param leftOrders The orders to be settled.
/// @param rightOrder The order to be settled.
/// @param leftSignatures The signatures of the order signed by maker.
/// @param rightSignature The signature of the order signed by maker.
function batchMatchOrders(
  LibOrder.Order[] memory leftOrders,
  LibOrder.Order[] memory rightOrders,
  bytes[] memory leftSignatures,
  bytes[] memory rightSignatures
) external;
```

**Logic**

Calling `batchMatchOrders` will perform the following steps:

