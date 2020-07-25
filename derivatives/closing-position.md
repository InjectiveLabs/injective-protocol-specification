- **Closing a Position**
    1. A trader must first currently have an open position of a given `quantity`, `contractPrice` and `margin`. 
        - $margin_{old}$ refers to the margin of the trader's position
        - $quantity_{old}$ refers to the quantity of the trader's position
        - $P_{contract_{old}}$ refers to the contract price of the trader's position.
    2. Trader must find one or more make `orders` of the same `marketID` and `direction` that he wishes to use to close his position and call `closePositionWithOrders(positionID, orders, quantity, signatures)`.  `positionID` refers to the positionID that the trader wishes to close, `quantity` refers to the quantity of contracts that the trader wishes to close, and `orders` and `signatures` refer to the make orders and make order signatures. 
        - $quantity_{new}$ refers to the total `quantity` of contracts being closed across the `orders`.
        - $P_{contract_{new}}$ refers to the weighted average of the contract price of the orders being used to close the position, i.e. $\frac{\sum\limits_{i \in orders} quantity_i \cdot P_{contract_i}}{quantity_{new}}$.
    3. Assuming each of the `orders` is valid and fillable with $quantity_i$, the makers of each `order` will have a new position established and the Trader's position will be closed. 

    Closing a position will first execute the funding payments for the position up to the current point in time and update the position's funding entry point to the current block timestamp. Doing so temporarily updates the margin for the position as follows:

    ### Perpetual Longs

    For Perpetual Longs, this yields the following new current margin value and PnL per contract: 

    $$margin_{curr}=margin_{old}-F_{entry}\cdot quantity_{old}$$

    $$PnL_{contract}=P_{contract_{new}}-P_{contract_{old}}$$

    - Since the trader wants to maximize the profit that he makes when he closes his position, he chooses `orders` with the highest $P_{contract_{new}}$ values.

    ### Perpetual Shorts

    For Perpetual Shorts, this yields the following new current margin value and PnL per contract:s

    $$margin_{curr}=margin_{old}+F_{entry}\cdot quantity_{old}$$

    $$PnL_{contract}=P_{contract_{old}}-P_{contract_{new}}$$

    Since the trader wants to maximize the profit that he makes when he closes his position, he chooses an `order` with the lowest $P_{contract_{new}}$ values.  

    For both longs and shorts, the net return to the Trader is then the following:

    $$return =quantity_{new}\cdot(\frac{margin_{curr}}{quantity_{old}}+PnL_{contract})$$

    The trader will have $quantity_{old} - quantity_{new}$ contracts of their position remaining with a margin of $margin_{new}=margin_{curr}-\frac{quantity_{new}\cdot margin_{curr}}{quantity_{old}}$. 

    # Restrictions on Closing Positions

    In order to prevent the possibility of the entire system becoming insolvent (unable to appropriately pay out traders), each long/short position has a minimum/maximum price at which it can be respectively closed based off its margin. 

    For both longs and shorts, the net return to the Trader is then the following:

    $return =quantity_{new}\cdot(\frac{margin_{curr}}{quantity_{old}}+PnL_{contract})$

    Simplifying to consider the case of just 1 contract and no funding (this generalizes for multiple contracts by simply setting $margin_{curr} = \frac{applyFunding(margin_{old})}{quantity_{old}}$), this is

    $return =margin_{curr}+PnL_{contract}$ which is

    **For longs:**

    - $return =margin_{curr}+P_{contract_{new}}-P_{contract_{old}}$

    **For shorts:**

    - $return =margin_{curr}+P_{contract_{old}}-P_{contract_{new}}$

    ## Liquidation Price Bound

    To decrease the risk of insolvency in the system (an implication of positions going beyond their bankruptcy price), positions are liquidated before the bankruptcy price is reached.

    This is accomplished by requiring each contract to have $P_{index} \cdot penalty$ amount of extra buffer in the position equity. Once this requirement is breached, a position is subject to liquidation (i.e. forcible closure) by any third party liquidator. Upon liquidation, any remaining equity in the position is split between the liquidator and the insurance pool.

    The price at which a position is subject to liquidation when the following conditions are breached:

    **For longs:**

    - $P_{index} \geq (1 + penalty)\cdot P_{contract_{old}}-margin_{curr}=P_{liquidation}$

    **For shorts:**

    - $P_{index} \leq (1 - penalty)\cdot P_{contract_{old}} + margin_{curr}=P_{liquidation}$

    Sanity check using $penalty = 0.05$

    - Suppose I went long at 8 USDT with 0.8 USDT as margin. This means my position is subject to liquidation when $P_{index}$ goes below (1+0.05)*8 -0.8 = 7.6 USDT.
    - Suppose I went short at 8 USDT with 0.8 USDT as margin. This means my position is subject to liquidation when $P_{index}$ goes above (1-0.05)*8 +0.8 = 8.4 USDT.

    Hence, when $P_{index} < (1 + penalty)\cdot P_{contract_{old}}-margin_{curr}$ for a long position or $P_{index} > (1 - penalty)\cdot P_{contract_{old}} + margin_{curr}$ for a short position, the position is subject to liquidation.

    At this point, any third party liquidator may forcibly close the position by creating a new position with the following price restriction on the new position contract price:

    **For longs:**

    - $P_{contract_{new}} \geq P_{index}$

    **For shorts:**

    - $P_{contract_{new}} \leq P_{index}$

    Liquidators are profit-seeking entities and hence will seek to make $|P_{contract_{new}}-P_{index}|$ as close to 0 as possible. For practical purposes (e.g. existing orderbook liquidity), this difference may not always equal 0.

    When the position is liquidated, the owner of the position will forfeit the margin that he originally deposited, and the $liquidatorRewardPercent$ percent of the $return$ from liquidation will go to the liquidator and $1-liquidatorRewardPercent$ percent of the return will go to the insurance pool.

    ### Bankruptcy Price Bound

    Closing a position must always result in a non-negative return, so we can impose the inequality constraint, $return \geq 0$, yielding the following price bound for $P_{contract_{new}}$ (ignoring transaction fees and liquidation fees):

    **For longs:**

    - $P_{contract_{new}} \geq P_{contract_{old}}-margin_{curr} = P_{bankruptcy}$

    **For shorts:**

    - $P_{contract_{new}} \leq P_{contract_{old}} + margin_{curr}= P_{bankruptcy}$

    Sanity check:

    - Suppose I went long at 8 USDT with 0.8 USDT as margin. This means I can’t close my position with a contract priced below 8-0.8 = 7.2 USDT.
    - Suppose I went short at 8 USDT with 0.8 USDT as margin. This means I can’t close my position with a contract priced above 8+0.8 = 8.8 USDT.