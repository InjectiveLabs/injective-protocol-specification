# Liquidation

* **Liquidating a Position - TODO**

  Currently, this model only supports isolated margined accounts \(i.e. 1 account holds 1 position\) and hence, an account being subject to liquidation means that its corresponding position is subject to liquidation. Once cross-margining across multiple positions is supported, we will change our nomenclature so that accounts will be considered to be subject to liquidation, not positions.

  For simplicity, we consider positions for liquidations.

  A perpetual `Position` held by a given `accountID` of a given `marketID` records a `direction`, `quantity`, `contractPrice`, `margin`, and `cumulativeFundingEntry`.

  Each `Market` records a given `initialMarginRatio`, `liquidationPenalty`, `indexPrice`, `currFundingTimestamp`, `fundingInterval`, `cumulativeFunding`, `makerTxFee`, `takerTxFee`, and a `relayerFeePercentage`.

```text
When an account's $NAV$ goes under 0, the account is subject to liquidation.

$NAV = \sum\limits\_{positions} \(margin + quantity\cdot \(NPV - minMargin\)\)$

For simplicity we consider the case when the account only holds 1 position. If the following liquidation condition is true, the account can be liquidated:

$$\frac{margin}{quantity} \leq P_{index}\cdot penalty - NPV$$

Hence for a perpetual long, this simplifies to:

$\frac{margin}{quantity} \leq P_{index}\cdot penalty - P_{index}+P_{contract} + F_{entry}$

$\frac{margin}{quantity} \leq \(-1+penalty\)\cdot P_{index}+P_{contract} + F\_{entry}$

For a perpetual short, this is:

$\frac{margin}{quantity} \leq P_{index}\cdot penalty - P_{contract}+P_{index}-F_{entry}$

$\frac{margin}{quantity} \leq \(1+ penalty\)\cdot P_{index} - P_{contract}-F\_{entry}$
```

* **Position Liquidation Price**

  Each position has a liquidation price and a bankruptcy price. When the index price reaches a position's liquidation price, the position can be liquidated. When the index price reaches a position's bankruptcy price, the position can be vaporized.

  A position's liquidation price is computed as follows:

  **Isolated Margin Long Position Liquidation Price**

  $margin + quantity \cdot \(P_{index} - P_{contract}-F_{entry} - P_{index}\cdot penalty\) \leq 0$

  $P_{index} \leq \frac{P_{contract}+F\_{entry}-\frac{margin}{quantity}}{1 - penalty}$

  **Isolated Margin Short Position Liquidation Price**

  $margin + quantity \cdot \(-P_{index}+P_{contract}+F_{entry} - P_{index}\cdot penalty\) \leq 0$

  $P_{index} \geq \frac{P_{contract}+F\_{entry}+\frac{margin}{quantity}}{1 + penalty}$

  **Cross Margin Long Position Liquidation Price**

  For a given position $i$, let $\mathrm{position value}\_i = margin + quantity\cdot \(NPV - minMargin\)$ for a given position.

  Hence, cross-margined positions are subject to liquidation when $\sum\limits\_{positions} \mathrm{position value}\_i \leq 0$ or equivalently when:

  $$P_{index_i}\leq \frac{ P_{contract_i} + F_{entry_i} - \frac{margin_i + \sum\limits_{j=positions \backslash \{i\}} \mathrm{position\ value}_j}{quantity_i} }{1-penalty_i}$$

  **Position Bankruptcy Price**

  Suppose you are liquidating a `position`

  Liquidating a position is similar to closing a position, except that

  Must pass in one or more `orders` to

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
  ) external {
  ```

