# Matching Orders

* **Matching Orders**

  Two orders of opposing directions can directly be matched if they have a negative spread. The conditions under which two orders \(a `leftOrder` and a `rightOrder`\) can be matched are the following:

  * Either the `leftOrder`'s  `makerAssetData` must contain the same  `marketID` as the `rightOrder`'s `takerAssetData` or the `leftOrder`'s  `takerAssetData` must contain the same  `marketID` as the `rightOrder`'s `makerAssetData` \(i.e. a long must be matched with a short\).
  * If the `leftOrder` is a Long, then the `rightOrder`'s `contractPrice` must be less than or equal to the `leftOrder`'s `contractPrice`. If the `leftOrder` is a Short, then the `rightOrder`'s `contractPrice` must be greater than or ~~~~equal to the `leftOrder`'s `contractPrice`.

    When a `leftOrder` is matched with a `rightOrder`,

    My req.order price can be found in `req.order.makerAssetAmount` and my req.order `quantity` can be found in `req.order.takerAssetAmount`.

  * get `marketID` from `makerAssetData` \(LONG\) or `takerAssetData` \(SHORT\).
  * 1. If my req.order is a SHORT, query the LONG orders \(orders where the `makerAssetData` contains the marketID\)
    2. If my req.order is a LONG, query the SHORT orders \(orders where the `takerAssetData` contains the marketID\)
  * Sort orders by price \(according to the two bulletpoints above\). 
    1. If req.order is SHORT, find the LONGs with the highest prices where the long price is greater than or equal to the short order price
       1. short price is 1.7, I want a list of orders of prices where \(1.9, 1.8, 1.74\)  in that order
    2. If req.order is LONG, find the SHORTs with the lowest prices where the short order price is less than or equal to the long order price
       1. If my long price is 1.7, I want a list of orders of \(1.2, 1.5, 1.69\) in that order
  * How long should this orders list be? The orders list should be the minimal set of orders needed to satisfy the req.order `quantity`. This is actually a bit involved to compute since the perfect way to do this would be to query the `getOrderRelevantStates(orders, signatures)` function on the Injective Futures contract and then iterate over the fillable orders summing up their `fillableTakerAssetAmounts` to find the minimal set of orders that would satisfy your quantity \(with perhaps some reasonable MAX\_ORDER\_LIST threshold to ensure that we dont run out of gas\). However, for simplicity purposes, for now we can naively just sum up the `order.takerAssetAmount` and use this as an optimistic estimate of the order's fillable quantity.
  * Call multiMatchOrders where leftOrders is a list of the sorted orders that we want to use, rightOrder is my req.order, leftSignatures is the signatures of the left orders, and rightSignature is my req.order signature

    ```go
    function multiMatchOrders(
         LibOrder.Order[] memory leftOrders,
         LibOrder.Order memory rightOrder,
         bytes[] memory leftSignatures,
         bytes memory rightSignature
     )
    ```

  * send at least 5 million gas as these calls are very gas intensive.

    **Suggested Implementation:** at the end of `postDerivativeOrder`, call `matchDerivativeOrder`, similar to how the end of `postOrder` calls `matchOrder`.

  * can query the same orders querier as regular orders,

* **Position/Account Maintenance**

  There are several things that a trader can do to maintain/view his account and positions.

  1. Query a list of one's `accountID`'s with `addressToAccountIDs[myAddress]`
  2. Query one's free deposits, unrestricted deposits and restricted deposits with `freeDeposits[myAddress]`, `unrestrictedDeposits[myAddress]`, `restrictedDeposits[myAddress]`
  3. Query the filled status of an order with `filled[orderHash]`. 
  4. Deposit more funds into one's account with `depositAccount(accountID, amount)` in order to prevent liquidation. 

