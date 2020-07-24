# Injective Perpetuals Protocol

A decentralized BitMEX. Injective Protocol facilitates the creation and bilateral trading of perpetual swaps.  

# Overview

This protocol enables traders to create, enter into, and execute decentralized perpetual swap contracts on any arbitrary market. 

Our design adopts an account balance model where all positions are publicly recorded with their respective accounts. Architecturally, the core logic is implemented on on-chain smart contracts while the order matching is done through on the Injective Chain (in an off-chain relay on-chain settlement fashion). 

In our system, users have **accounts** which manage their **positions** in one or more futures **markets.** Each futures market specifies the bilateral futures contract parameters and terms, including most notably the margin ratio and oracle. Buyers and sellers of contracts in a futures market coordinate off-chain and then enter into contracts through an on-chain transaction. At all times, the payout (NPV) of long positions are balanced with corresponding short positions. The positions held by an account are subject to liquidation when the NAV of the account becomes negative. 

While a market is live, market-wide actions may affect all positions in the market such as dynamic funding rate adjustment (to balance market long/shorts) as well as clawbacks in rare scenarios. 

- **Key terms**

    **Perpetual Swap** - A derivative providing exposure to the value of an underlying asset which mimics a margin-based spot market and has no expiry/settlement. 

    **CFD** - Contract for difference which is a derivative stipulating that the sellers will pay the buyers the difference between the current value of an asset and its value at the contract's closing time.

    **Address** - A secp256k1 based address. It's important to differentiate an address from an account in the futures protocol.

    **Account** - An address can own one or more accounts which hold positions. Isolated margin positions are each held by their own account while cross margined positions are held by the same account.

    **Deposits** - The value of an account's aggregate deposits denominated in a specified base currency (e.g. USDT). 

    **Market** - A market is defined by its supported base currency, oracle, and minimum margin ratio / margin requirement. The market may also support a unique funding rates calculation, which is a distinctive feature of a perpetual swap futures market. 

    **Direction** - Long or Short.

    **Index Price** - The reference spot index price of the underlying asset that the derivative contract derives from. This value is obtained from an oracle. 

    $$P_{index}=\mathrm{index\ price\ of\ underlying}$$

    **Contract Price -** The value of the futures contract the position entered/created. This is purely market driven and is set by individuals, independent of the oracle price.

    $$P_{contract}=\mathrm{contract\ price}$$

    **Position** - Executing a contract creates two opposing positions (one long and one short) which  reflects the ownership conditions of some **Quantity** of contracts. 

    A cross margined account can hold multiple positions while an isolated margined account holds one position.

    **Margin** - Margin refers to the amount of base currency that a trader posts as collateral for a given position. 

    **Contract** - A contract refers to the single unit of account for a position in a given direction in a given market. There are two sides of a contract (long or short) that one can enter into. 

    - **Perpetual Swap example**

        As an example, consider the mBTCUSDT perpetual swap contract. 1 contract represents the USDT value of exposure to 1 mBTC (0.001 BTC) which has an **Index Price** of 8 USDT when the spot price of Bitcoin is $8000. 

        Two parties (with Larry representing a Long, and the Sally representing a Short) may enter into a contract with a given **Contract Price**, where the Long purchases synthetic Long exposure to mBTC and the Short purchases synthetic Short exposure to mBTC. 

        Suppose Larry and Sally both decide to enter into a **Perpetual** contract with a **Funding Period** of 12 hours with a **Contract Price** of 8 USDT and each decide to use 8 USDT of their **Deposits** as **Margin** (1x leverage, fully collateralized). 

        At the end of the funding period, suppose the **Funding Fee** is 0.01. Since the Funding Fee is positive, Longs must pay Shorts 0.01 USDT for every contract they own. Note: due to transaction costs associated with this model, the implementation of the funding is done differently and is discussed below but yields the same net result as described in this example. 

    - **CFD Example**

        As an example, consider the mBTCUSDT perpetual swap contract. 1 contract represents the USDT value of exposure to 1 mBTC (0.001 BTC) which has an **Index Price** of 8 USDT when the spot price of Bitcoin is $8000. 

        Two parties (with Larry representing a Long, and the Sally representing a Short) may enter into a contract with a given **Contract Price**, where the Long purchases synthetic Long exposure to mBTC and the Short purchases synthetic Short exposure to mBTC. 

        Suppose Larry and Sally both decide to enter into a **CFD** contract with a **Contract Price** of 8 USDT and each decide to use 8 USDT of their **Deposits** as collateral (1x leverage, fully collateralized). 

        Unlike a Perpetual contract, a **CFD** has an expiration time when the contract is settled. At settlement, the Long will pay the Short the difference between the **Contract Price** and the **Index Price** of the underlying (mBTC) at expiration.

        Suppose that upon settlement, the Index Price of mBTC has risen to 10 USDT. Since **Contract Price - Index Price** = -2, the Short will pay the Long 2 USDT. Had the Index Price of mBTC been 6, the Long would have paid the Short 2 USDT. 

        Perpetual swaps do not have an expiration time but rather are settled continuously with a **Funding Rate**, the details of which are specified below. 

    The above examples were for a single **Contract**. For multiple contracts, **Quantity** is used. 

    $$quantity = \mathrm{quantity\ of\ contracts}$$

    **Order** - A cryptographically signed message expressing an unilateral agreement to enter into a position under certain specified terms (i.e. make order or take order). 

    **Funding** - Funding refers to the periodic payments exchanged between the traders that are long or short of a perpetual contract at the end of every funding period (e.g. every 8 hours). When the funding rate is positive, longs pay shorts. When negative, shorts pay longs. 

    **Funding Rate** - The funding rate value determines the funding fee that positions pay and is based on the difference between the price of the contract in the perpetual swap markets and spot markets. Further details can be found below. 

    [Funding Rate Calculation](https://www.notion.so/Funding-Rate-Calculation-59be6b63293241d4b88d68487441b9f3)

    **Funding Fee** - The funding fee $f$ refers to the funding payment that is made for a **single contract** in the market for a given epoch $i$ and cumulative funding $F$ refers to the cumulative funding for the contract since position entry. 

    $$f_{i} = \mathrm{unit\ funding\ fee\ at\ epoch\ }i$$

    $$F_{entry}= \sum \limits_{i=entry}^{current-1} f_i$$

    Hence, $f_1=0.01$ means that open long contracts at this time will pay 0.01 units of base currency and $f_2 = -0.03$ means that shorts should pay 0.03 units of base currency. 

    **NPV** - Net Present Value of a single contract. For long and short perpetual contracts, the NPV calculation is:

    $$NPV_{perpetual\ long}= P_{index}-P_{contract} - F_{entry}$$

    $$NPV_{perpetual\ short} = P_{contract}-P_{index}+F_{entry}$$

    **Liquidation -** When an account's **NAV** becomes negative, all of its positions are subject to liquidation, i.e. the forced closure of the position due to a minimum collateral requirement being breached. 

    **Liquidation Penalty** - The liquidation penalty is a fixed percentage defining the value of the liquidated position that is paid out. Each market can define its own liquidation penalty (e.g. 5% in the example below) but every market's liquidation penalty must be greater than the global minimum liquidation penalty. 

    $$penalty = 0.05$$

    **Minimum Margin** - The minimum margin refers to the minimum amount of margin that a single contract must have at all times and is defined as:

    $$minMargin = P_{index} \cdot  penalty$$

    **Initial Margin** -  The initial minimum margin defines an additional component of the minimum margin requirement that every position in a given market must satisfy when first created. This requirement is stricter than the minimum margin requirement and exists in order to prevent unnecessary liquidation. 

    Each market can define its own initial margin (e.g. 0.5 in the example below) but every market's margin ratio must be greater than the global minimum margin ratio. 

    $$initialMargin=penalty+initialMarginFactor=0.05$$

    Upon position creation, each long and short being matched must satisfy the following:

    $$\frac{margin_{long}}{quantity} \geq P_{contract}\cdot initialMargin$$

    $$\frac{margin_{short}}{ quantity} \geq (2\cdot P_{index}-P_{contract})\cdot initialMargin$$

    By doing so the net margin deposited is invariant with respect to the **Contract Price** (which is determined by by the p2p market) and only changes with respect to the **Index Price**, satisfying the following:

    $$\frac{margin_{long}+margin_{short}}{quantity} \geq 2\cdot P_{index}\cdot initialMargin$$

    See further discussion here:

    [Initial Margin Requirements](https://paper.dropbox.com/doc/Initial-Margin-Requirements--A1JdzhXp4deDBGCLYMpVvlpIAQ-vs8H7CzCjrv2nId7JVPaa)

    **NAV** - Net asset value of an account, important to differentiate from NPV. Used mainly for liquidation or dispute calculation. Follows the following formula:

    $$NAV = \sum_{positions} (margin + quantity\cdot (NPV - minMargin))$$

    margin +quantity *(Pindex- Pcontract - Pindex * penalty)

    margin +quantity *(Pcontract-Pindex - Pindex * penalty)

    **Liquidation Fee** - The fee that a liquidated position pays. The proceeds from liquidation fees are split between the liquidator and insurance fund.

    $$liquidationFee =penalty\cdot P_{index} \cdot quantity$$

    **Clawback** - A clawback event occurs when a threshold number of accounts cannot be liquidated with a net-zero final payout. In practice, this happens when a chain reaction of unidirectional liquidation cannot be liquidated with the position margin due to volatility or lack of liquidity. 

# Architecture

The Injective Futures Protocol uses off-chain order relay with on-chain settlement. In this approach, cryptographically signed orders are broadcast to the decentralized orderbook on the Injective Chain. Then, an interested counterparty may inject one or more of these orders into the Injective Futures Protocol contract to execute and settle trades directly to the blockchain. 

# **Orders**

We leverage the [0x Order Message format](https://github.com/0xProject/0x-protocol-specification/blob/master/v3/v3-specification.md#order-message-format) for the external interface to represent a make order for a derivative position, i.e. a cryptographically signed message expressing an agreement to enter into a derivative position under specified parameters. 

- **Order Message Format**

    An order message consists of the following parameters:

    - `address makerAddress` - Address that created the order.
    - `address takerAddress` -  Empty.
    - `address feeRecipientAddress` - Address of the partial recipient of the order transaction fee.
    - `address senderAddress` - Empty.
    - `uint256 makerAssetAmount` - The `contractPrice`, i.e. the price of 1 contract denominated in base currency.
    - `uint256 takerAssetAmount` - The `quantity` of contracts the maker seeks to obtain.
    - `uint256 makerFee` -  The amount of `margin` of base currency the maker would like to post/risk for the order.
    - `uint256 takerFee` - Empty.
    - `uint256 expirationTimeSeconds` - Timestamp in seconds at which order expires.
    - `uint256 salt` -  Arbitrary number to facilitate uniqueness of the order's hash.
    - `bytes makerAssetData` - The first 32 bytes contain the `marketID` of the market for the position if the order is LONG, empty otherwise.  Right padded with 0's to be 36 bytes
    - `bytes takerAssetData` - The first 32 bytes contain the `marketID` of the market for the position if the order is SHORT, empty otherwise. Right padded with 0's to be 36 bytes
    - `bytes makerFeeAssetData` - Empty.
    - `bytes takerFeeAssetData` - Empty.

    In a given perpetual market specified by `marketID`, an order encodes the willingness to purchase `quantity` contracts in a given direction (long or short) at a specified `contractPrice` using a specified amount of `margin` of base currency as collateral. 

- **Prerequisites**
    1. The user must find an active market which he wants to participate in and must have some quantity of the market’s supported base currency. This base currency willl be used as margin in order to create valid orders which can then be filled to enter into positions. 
    2. The user must have the ERC-20 approval of the base currency token set to the futures contract address. 
- **Creating an Order**
    1. Maker creates an order and signs it off-chain.
    2. Maker sends the order to the decentralized order book on the Injective Chain. 

    In order for a given order to be valid, it must satisfy the following margin property:

    $$ margin \geq quantity \cdot P_{contract} \cdot (initialMargin + penalty)$$

    - Expressed in the order message format, this is `makerFee >= takerAssetAmount * makerAssetAmount * (market.initialMarginRatio + market.liquidationPenalty)`
- **Filling an Order**
    1. Taker discovers the make order from the Injective Chain and decides to fill some quantity of the order. This means that he wishes to enter into a position in the opposite direction of the make order at the Contract Price specified in the make order for some desired quantity of contracts using his desired amount of margin as collateral.
    2. Taker calls  `fillOrder(order, quantity, margin, signature)` on the futures smart contract. This will immediately enter the workflow of establishing two positions - one for the maker and one for the taker. 
    3. By doing so, the futures contract transfers `margin` amount of the base currency specified by the make order and take order using ERC-20 the `transferFrom` call and deposits the tokens into their balance. 
    4. The positions are attempted to be created for both parties. If the following conditions succeed, then the positions are created. If any one of these checks fail the entire transaction is reverted.

        $$\frac{margin_{long}}{quantity} \geq P_{contract}\cdot initialMargin$$

        $$\frac{margin_{short}}{ quantity} \geq (2\cdot P_{index}-P_{contract})\cdot initialMargin$$

        - If any of the orders being used was filled before, the `quantity` being taken must not be greater than what is remaining.

    Now, suppose a maker creates a huge make order and a taker decides to partial fill it. If it is the first time that the maker and taker are trading, by default, the entire maker's deposits are used as margin for this position, and the same is true for the taker. 

- **Matching Orders**

    Two orders of opposing directions can directly be matched if they have a negative spread. The conditions under which two orders (a `leftOrder` and a `rightOrder`) can be matched are the following: 

    - Either the `leftOrder`'s  `makerAssetData` must contain the same  `marketID` as the `rightOrder`'s `takerAssetData` or the `leftOrder`'s  `takerAssetData` must contain the same  `marketID` as the `rightOrder`'s `makerAssetData` (i.e. a long must be matched with a short).
    - If the `leftOrder` is a Long, then the `rightOrder`'s `contractPrice` must be less than or equal to the `leftOrder`'s `contractPrice`. If the `leftOrder` is a Short, then the `rightOrder`'s `contractPrice` must be greater than or ~~~~equal to the `leftOrder`'s `contractPrice`.

    When a `leftOrder` is matched with a `rightOrder`, 

    My req.order price can be found in `req.order.makerAssetAmount` and my req.order `quantity` can be found in `req.order.takerAssetAmount`. 

    1. get `marketID` from `makerAssetData` (LONG) or `takerAssetData` (SHORT). 
    2. 
        1. If my req.order is a SHORT, query the LONG orders (orders where the `makerAssetData` contains the marketID)
        2. If my req.order is a LONG, query the SHORT orders (orders where the `takerAssetData` contains the marketID)
    3. Sort orders by price (according to the two bulletpoints above). 
        1. If req.order is SHORT, find the LONGs with the highest prices where the long price is greater than or equal to the short order price
            1. short price is 1.7, I want a list of orders of prices where (1.9, 1.8, 1.74)  in that order
        2. If req.order is LONG, find the SHORTs with the lowest prices where the short order price is less than or equal to the long order price
            1. If my long price is 1.7, I want a list of orders of (1.2, 1.5, 1.69) in that order
    - How long should this orders list be? The orders list should be the minimal set of orders needed to satisfy the req.order `quantity`. This is actually a bit involved to compute since the perfect way to do this would be to query the `getOrderRelevantStates(orders, signatures)` function on the Injective Futures contract and then iterate over the fillable orders summing up their `fillableTakerAssetAmounts` to find the minimal set of orders that would satisfy your quantity (with perhaps some reasonable MAX_ORDER_LIST threshold to ensure that we dont run out of gas). However, for simplicity purposes, for now we can naively just sum up the `order.takerAssetAmount` and use this as an optimistic estimate of the order's fillable quantity.

    4. Call multiMatchOrders where leftOrders is a list of the sorted orders that we want to use, rightOrder is my req.order, leftSignatures is the signatures of the left orders, and rightSignature is my req.order signature

    ```go
    function multiMatchOrders(
            LibOrder.Order[] memory leftOrders,
            LibOrder.Order memory rightOrder,
            bytes[] memory leftSignatures,
            bytes memory rightSignature
        )
    ```

    - send at least 5 million gas as these calls are very gas intensive.

    **Suggested Implementation:** at the end of `postDerivativeOrder`, call `matchDerivativeOrder`, similar to how the end of `postOrder` calls `matchOrder`. 

    - can query the same orders querier as regular orders,
- **Market Orders**

    `marketOrders(orders, quantity, margin, signatures)`

    iterate through orders:

    find fi

- **Position/Account Maintenance**

    There are several things that a trader can do to maintain/view his account and positions.

    1. Query a list of one's `accountID`'s with `addressToAccountIDs[myAddress]`
    2. Query one's free deposits, unrestricted deposits and restricted deposits with `freeDeposits[myAddress]`, `unrestrictedDeposits[myAddress]`, `restrictedDeposits[myAddress]`
    3. Query the filled status of an order with `filled[orderHash]`. 
    4. Deposit more funds into one's account with `depositAccount(accountID, amount)` in order to prevent liquidation. 
- **Closing a Position**
    1. A trader must first currently have an open position of a given `quantity`, `contractPrice` and `margin`. 
        - $margin_{old}$ refers to the margin of the trader's position
        - $quantity_{old}$ refers to the quantity of the trader's position
        - $P_{contract_{old}}$ refers to the contract price of the trader's position.
    2. Trader must find one or more make `orders` of the same `marketID` and `direction` that he wishes to use to close his position and call `closePositionWithOrders(positionID, orders, quantity, signatures)`.  `positionID` refers to the positionID that the trader wishes to close, `quantity` refers to the quantity of contracts that the trader wishes to close, and `orders` and `signatures` refer to the make orders and make order signatures. 
        - $quantity_{new}$ refers to the total `quantity` of contracts being closed across the `orders`
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

- **Liquidating a Position - TODO**

    Currently, this model only supports isolated margined accounts (i.e. 1 account holds 1 position) and hence, an account being subject to liquidation means that its corresponding position is subject to liquidation. Once cross-margining across multiple positions is supported, we will change our nomenclature so that accounts will be considered to be subject to liquidation, not positions. 

    For simplicity, we consider positions for liquidations. 

    A perpetual `Position` held by a given `accountID` of a given `marketID`  records a `direction`, `quantity`, `contractPrice`, `margin`, and `cumulativeFundingEntry`.  

    Each `Market` records a given `initialMarginRatio`, `liquidationPenalty`, `indexPrice`, `currFundingTimestamp`, `fundingInterval`, `cumulativeFunding`, `makerTxFee`, `takerTxFee`, and a `relayerFeePercentage`. 

    - Market and Position Struct Definitions

        ```
        struct Market {
            bytes32 marketID;
            string ticker;
            address oracle;
            uint256 initialMarginRatio;        // /1000
            uint256 liquidationPenalty; // /1000
            uint256 indexPrice;
            uint256 currFundingTimestamp; // the current funding timestamp
            uint256 fundingInterval; // defines the interval in seconds by which the currFundingTimestamp increments
            int256 cumulativeFunding;   //Stored based on one contract. /10^6
            PermyriadMath.Permyriad makerTxFee;   // transaction maker fee
            PermyriadMath.Permyriad takerTxFee;   // transaction taker fee
            PermyriadMath.Permyriad relayerFeePercentage;   // transaction relayer fee percentage
        }
        ```

        ```
        struct Position {
            // owner of the position
            bytes32 accountID;
            // marketID of the position
            bytes32 marketID;
            // direction of the position
            Direction direction;
            // quantity of the position
            uint256 quantity;
            // contractPrice of the position
            uint256 contractPrice;
            // Net present value of the position
            int256 NPV;
            uint256 minMargin;
            uint256 margin; // used to be amount provided
            int256 cumulativeFundingEntry; // Just for perpetuals. The epoch of fundingRate this position entered in.
        }

        ```

    When an account's $NAV$ goes under 0, the account is subject to liquidation. 

    $NAV = \sum\limits_{positions} (margin + quantity\cdot (NPV - minMargin))$

    For simplicity we consider the case when the account only holds 1 position. If the following liquidation condition is true, the account can be liquidated:

    $$\frac{margin}{quantity} \leq P_{index}\cdot penalty - NPV$$

    Hence for a perpetual long, this simplifies to:

    $\frac{margin}{quantity} \leq P_{index}\cdot penalty - P_{index}+P_{contract} + F_{entry}$

    $\frac{margin}{quantity} \leq  (-1+penalty)\cdot P_{index}+P_{contract} + F_{entry}$

    For a perpetual short, this is:

    $\frac{margin}{quantity} \leq P_{index}\cdot penalty - P_{contract}+P_{index}-F_{entry}$

    $\frac{margin}{quantity} \leq (1+ penalty)\cdot P_{index} - P_{contract}-F_{entry}$

    - **Position Liquidation Price**

        ### Position Liquidation Price

        Each position has a liquidation price and a bankruptcy price. When the index price reaches a position's liquidation price, the position can be liquidated. When the index price reaches a position's bankruptcy price, the position can be vaporized. 

        A position's liquidation price is computed as follows:

        **Isolated Margin Long Position Liquidation Price**

        $margin + quantity  \cdot (P_{index} - P_{contract}-F_{entry} - P_{index}\cdot penalty) \leq 0$ 

        $P_{index} \leq \frac{P_{contract}+F_{entry}-\frac{margin}{quantity}}{1 - penalty}$

        **Isolated Margin Short Position Liquidation Price**

        $margin + quantity  \cdot (-P_{index}+P_{contract}+F_{entry} - P_{index}\cdot penalty) \leq 0$ 

        $P_{index} \geq \frac{P_{contract}+F_{entry}+\frac{margin}{quantity}}{1 + penalty}$

        **Cross Margin Long Position Liquidation Price**

        For a given position $i$, let $\mathrm{position\ value}_i = margin + quantity\cdot (NPV - minMargin)$ for a given position.

        Hence, cross-margined positions are subject to liquidation when $\sum\limits_{positions} \mathrm{position\ value}_i \leq 0$ or equivalently when:

        $$P_{index_i}\leq \frac{ P_{contract_i} + F_{entry_i} - \frac{margin_i + \sum\limits_{j=positions \backslash \{i\}} \mathrm{position\ value}_j}{quantity_i} }{1-penalty_i}$$

        ### Position Bankruptcy Price

    Suppose you are liquidating a `position` 

    Liquidating a position is similar to closing a position, except that 

    Must pass in one or more `orders`  to 

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

- **Example**

    (A BIT INACCURATE SINCE THE SHORT IS UNREALISTIC UNLESS WE SUPPORT NEGATIVE PRICES WHICH WE DONT)
    Suppose 1 mBTC contract represents 1/1000 of a BTC contract. Say BTC is 8000 USDT now. 
    I want long exposure to 0.5 BTC, BUT I'm cheap and want to see if some idiot will give me exposure to 0.5 BTC (500 mBTC) at a rate of 10 USDT/BTC (so 10 USDT/ 1000 mBTC which equals 0.01 USDT/mBTC). 

    Market details: 

    - 1 contract = 1mBTC = 1/1000 BTC
    - Max leverage = 10x
    - liquidation penalty = 5%
    - minMargin = 5%

    `makerAssetAmount` = contract price of the 1mBTC = 0.01 USDT/mBTC

    `takerAssetAmount` = quantity of contracts

    `uint256 makerFee` -  The amount of margin of base currency the maker would like to post/risk for the order. 

    So I create an order where

    - makerAddress = cheap dude address
    - `makerAssetAmount` = 0.01 USDT/mBTC (a 800x "discount" - a huge bargain!)
    - `takerAssetAmount` = 500 mBTC
    - `makerFee` = 0.5 USDT

    When a relayer receives this order, the relayer should check whether the account is sufficiently collateralized. He can check by looking at the account's margin.

    - contract price * quantity =  0.01 * 500 = $5
    - Cheap guy USDT margin in "my account ID"= $0.50
    - min collateral ratio = minMargin + liquidation penalty = 10%
    - The cheap guy can have any amount of margin in his account, so long as it satisfies the min collateral ratio of the contract value, aka 0.1*5 = $0.50

    $$deposit  \geq quantity \cdot P_{contract} \cdot (marginRatio + penalty)$$

    $$deposit \geq 500 \cdot 0.01 \cdot (0.05+0.05) = 0.5$$

    $$deposit =order.makerFee \geq 0.5$$

    - Cheap guy collateral ratio = margin / contract value = $0.5 / $5 = 10%
    - FYI, the relayer should consistently check the collateralization of cheap guy's make order long as its on the orderbook. Because if the USDT value or other margin he's using (INJ for example) has a sudden drop, the position might be no longer sufficiently collateralized.

    The Oracle provides the index price, aka the value denominated in baseCurrency of 1 mBTC contract. 

    - `getPrice` returns 8 USDT

    $$P_{index}=8$$

    $$NPV_{perpetual\ long}= quantity \cdot (P_{index}-P_{contract} - (F_{current} - F_{entry})) $$

    $$NPV_{account}=NPV_{perpetual\ long}= 500 \cdot (8-0.01 - (0 - 0)) = 3995$$

    In this example the Account NPV equals the NPV of the perpetual long since the maker only has one position. 

    NPV (account) assume if filled = $8 - $0.01 = $7.99

    $$NAV =Deposits +NPV_{account}- \sum_{positions} (minMargin)$$

    $$NAV = 0.5 + 3995 - 8\cdot 500\cdot  (0.05 + 0.05) = 3595.5$$

    NAV = $7.24

    Contract Price calculation (basically assume that by the time the contract is executed, the oracle price = contract price):

    Contract price= $0.01

    NAV= $0.5 + 0 - (10%) * $0.01 * 500

    NAV= 0

    The futures protocol goes by the oracle price calculation, but the relayer should calculate both. Because the relayer should operate under the assumption that by the time the order is taken, the oracle price is approximately the same as contract price. So both NAV calculations have to be above or = 0. 

    The cheap guy's position is sufficiently collateralized, therefore the relayer accepts the cheap guy's position. 

    For the idiot taker, we must have

    0 ≤ NAV = Deposits + NPV_account - sum(minMargins)

    -1*Deposits ≤ quantity * (P_contract - P_index) - (P_index * quantity)*(marginRatio + penalty)

    Deposits ≥ -1*quantity * [(P_contract - P_index) - (P_index)*(marginRatio + penalty)]

    Deposits ≥ quantity * [P_index - P_contract + P_index*(marginRatio + penalty)]

    $$deposit \geq quantity \cdot (P_{index}-P_{contract}+P_{index}\cdot(marginRatio + penalty))$$

    $$deposit \geq 500 \cdot (8-0.01+8\cdot(0.05+0.05)) = 4395$$

    $$deposit +quantity \cdot P_{contract} \geq quantity \cdot P_{index}+quantity*P_{index}\cdot(marginRatio + penalty)$$

    ### Make Order 2 ("Take Order" from idiot)

    This denotes a make order for short exposure to 0.5 BTC ($4000 value). This make order says I will give makerAssetAmount = ~3995 USDT or whatever the lowest amount so that the NAV for the position of $10 at $8000 price oracle to be 0 to get short exposure on 0.5 BTC at $10. In practice, 

    Selling 1 mBTC for 0.01 baseCurrency

    - `makerAssetAmount` = 0.01 USDT/mBTC
    - `takerAssetAmount` = 500
    - `makerFee` ≥ ???

    Once the idiot provides the fillOrder, the relayer can check collateralization on the idiot or send it straight to futures protocol. This depends on relayer's responsibility

    How much collateral the cheap guy needs:

    Oracle Price calculation:

    $$NPV_{perpetual\ short} = quantity \cdot (P_{contract}-P_{index}+(F_{current}-F_{entry}))$$

    $$NPV_{perpetual\ short} = 500 \cdot (0.01-8+(0-0)) =-3995$$

    NPV (account) assume if filled = -3995

    0 = margin deposit - $7.99*500 - (10%)*$8*500

    Margin deposit = 3995+400

    = $4395P_{contract}

    $$NAV =Deposits +NPV_{account}- \sum_{positions} (minMargin)$$

    $$NAV =Deposits -3995- 8\cdot 500\cdot (0.05+0.05)$$

    NAV = 0

    In order to execute successfully, both parties' NAVs must always be greater than 0. This means that the idiot must have at least -1*(-3995-8*500*(0.05+0.05) = 4395 USDT in his deposits to use as margin. 

    Contract Price calculation (basically assume that by the time the contract is executed, the oracle price=contract price):

    Contract price= $0.01

    NAV= $0.5 + 0 - (10%) * $0.01 * 500

    NAV= 0

    In order for both positions to be sufficiently collateralized, the idiot needs to post $4395 as margin(lol). It's worse than 1x leverage because the idiot has to also post collateral for margin deposit. So he's effectively over collateralizing his position.

[On Closing Positions and Liquidations](https://www.notion.so/On-Closing-Positions-and-Liquidations-f92ff360a0f54576bfe1e55a13d42dbf)

[Smart Contract Implementation](https://www.notion.so/Smart-Contract-Implementation-3130ec499e5748bba65962ad9d3a1eb7)

## Transaction Fees

The business model for traditional derivatives exchanges (e.g. BitMEX, Binance, FTX, etc.) is to take a transaction fee from the each trade. However, because Injective Protocol is designed as a fully decentralized network, we do not follow this approach. 

Instead, transaction fees paid in Injective Protocol are used for two purposes. 

1. As compensation for relayers for driving liquidity to the protocol
2. As auction collateral for a "buyback and burn" process. In this token economic model, transaction fees collected over a fixed period of time (e.g. 2 weeks) are aggregated and then auctioned off to the public for the INJ token. The INJ received from this auction is then permanently burned. 

There are two types of transaction fees that should be introduced: **maker fees** (for limit or "maker" orders) and **taker fees** (for market or "taker" orders). For both maker and taker fees, transaction fees are to be extracted from the **notional value** of the trade. 

### Maker Fees

Maker fees are fees that makers of limit orders pay. These are the fees paid by the order makers (found in the `makerAddress` parameter of each order). Limit orders (i.e. the orders from which maker fees apply) are `leftOrders` and `rightOrder` in the `multiMatchOrders` function, `orders` in the `marketOrders` function, `order` in the `fillOrder` function, `order` in the `closePosition` function, and `orders` in the `closePositionWithOrders` function. 

**Notional Value of Maker Orders**

Recall that within a given perpetual market, an order encodes the willingness to purchase up to `quantity` contracts in a given direction (long or short) at a specified `contractPrice` using a specified amount of `margin` of base currency as collateral. 

The notional value of a single contract is simply the `contractPrice` (or $P_{contract}$).  Hence, the notional value of $n$ contracts is simply `n * contractPrice`. Note that orders can be partially filled so `n` must be less than or equal to `quantity`. 

This the calculation for notional value that you will need to use for`leftOrders` and `rightOrder`* in `multiMatchOrders`, `orders` in the `marketOrders` function, `order` in the `fillOrder` function, `order` in the `closePosition` function, and `orders` in the `closePositionWithOrders` function. 

* Note: the `contractPrice` for the `rightOrder` should be the weighted average `contractPrice` of the `leftOrders`. 

### Taker Fees

Taker fees are the fees paid by the `msg.sender` (the taker) in the `fillOrder` and `marketOrders` calls. 

**Notional Value for Taker Orders**

The notional value calculation in `fillOrder` for the taker is simply `notional = quantity * contractPrice`. 

The notional value calculation in `marketOrders` is the sum of the notional values for the quantity of contracts taken in each of the individual orders i.e. `notional = quantity_1 * contractPrice_1 + quantity_2 * contractPrice_2 + ... + quantity_n * contractPrice_n`. 
Note that $\sum \limits_{i=1}^{n} quantity_i$  must equal `quantity`. 

The notional value calculation in `closePosition` and `closePositionWithOrders` for the taker (i.e. the closer/msg.sender) is `notional = quantity * avgContractPrice` (`avgContractPrice` is already defined in the contract). 

**Maker Orders Transaction Fees**

Currently, whenever `n` contracts of a maker `order` are executed, the proportional amount of `margin` (where `margin = order.makerFee * n / order.takerAssetAmount`) is transferred from the user's base currency ERC-20 balance to the perpetuals contract. 

With the introduction of the maker order fee, executing `n` contracts of the `order` should result in the trader to post `margin + txFee` where `txFee = notional * MAKER_FEE_PERCENT / 10000` where `notional = n * order.makerAssetAmount`and where `MAKER_FEE_PERCENT` refers to the digits of the maker fee percentage scaled by 10000 (i.e a 0.15% maker fee would be 15). 

**Taker Orders Transaction Fees**

Similarly, with the introduction of the taker order fee, executing `n` contracts of the `order` should result in the trader posting `margin + txFee` where `txFee = notional * TAKER_FEE_PERCENT / 10000`  where `TAKER_FEE_PERCENT` refers to the digits of the taker fee percentage scaled by 10000 (i.e a 0.25% taker fee would be 25) and where notional is described in the **Notional Value for Taker Orders** section.

**Transaction Fee Distribution**

For a given `order` with a transaction fee of `txFee`, if the `feeRecipientAddress` is defined, `txFee * RELAYER_FEE_PROPORTION / 100`  should be distributed to the `feeRecipientAddress`'s balance. and the remainder should be distributed to the auction contract's balance. Note: "distributed to balance" in this case does not mean an ERC-20 transfer but rather incrementing the recipient's internal balance in the perpetuals contract from which the recipient can withdraw from in the future (this withdrawal functionality already exists). 

## Basic Futures Walkthrough

OUTDATED, NEED TO UPDATE

- Formulas Reference

    For reference, the fomulas used are provided below:

    $$minMargin = (marginRatio + penalty) \cdot P_{index} \cdot quantity$$

    $$NPV_{long\ position}= - NPV_{short\ position}=quantity \cdot (P_{index}-P_{contract})$$

    $$NPV_{account}=\sum_{positions}{NPV_{position}} $$

    $$NAV =deposit +NPV_{account}-\sum_{positions} (minMargin)$$

- Simple Scenario Walkthrough

    For simplicity, we denote values in USD as a proxy for `baseCurrency`. For cases when the INJ token is used as the base currency, denoting the values with dollar signs (e.g. $20) should be replaced with INJ (e.g. 20 INJ). We also consider Alice and Bob's position NPVs as their accounts NPVs since Alice and Bob each only have 1 position. 

    Suppose the price of 1 ETH = $100. 

    1. Market for fETH (future ETH) created with: 
        - Margin ratio of 10%
        - Liquidation penalty of 5%
        - Expiration date in 1 month
        - Contract Price of $100 (i.e. 1 fETH contract represents the value of 1 ETH)
    2. Alice and Bob both deposit $20 of `baseCurrency` into the Accounts contract.
    - **Parameter Values**

        Alice's NAV = +$20
        Bob's NAV = +$20 

    3. Alice creates a signed Order Message to enter 1 long position for a fETH and sends this to the Injective Sidechain. 

    4. Bob sees Alice's order and decides to fill it, establishing his short position and Alice's long position. 

    - **Parameter Values**

        Alice's NPV = quantity*(Pindex - Pcontract) = 1*(100 - 100) = $0
        Alice's NAV = deposits + NPV - minMargin =  20 - 0 - (0.10 + 0.05)*100*1 = +$5
        Bob's NPV = quantity*(Pcontract - Pindex) = 1*(100 - 100) = $0
        Bob's NAV = deposits + NPV - minMargin =  20 - 0 - (0.10 + 0.05)*100*1 = +$5

    5. The price of ETH drops to $90. 

    - **Parameter Values**

        Alice's NPV = 1*(90 - 100) = -$10
        Alice's NAV =  20 - 10 - (0.10 + 0.05)*90*1 = -$3.50
        Bob's NPV = -1*(100 - 110) = +$10
        Bob's NAV =  20 + 10 - (0.10 + 0.05)*90*1 = +$16.50

    Alice's account is now subject to liquidation as her NAV is negative.

    6. To liquidate Alice, Carol a third-party liquidator opens a counterbalancing short position of 1 fETH to net out her long position, which first requires Carol to find a long order from the orderbook and fill it. Let Carol also have $20 in her account deposits. 

    - **Step 1: Parameter Values**

        Alice's NPV = 1*(90 - 100) = -$10
        Alice's NAV =  20 - 10 - (0.10 + 0.05)*90*1 = -$3.50
        Carol NAV = 20 + 0 - (0.10 + 0.05)*90 = +$6.50

    In the liquidation, Alice will have the following amount refunded to her 

    The liquidator will earn the following liquidation proceeds which equals 0.05*90*1 = +$4.50

    $$LiquidationProceeds= penalty\cdot P_{index}*quantity$$

    while Alice has the following liquidation remains to her which equals 20 - 10 - 0.05*90*1 = $5.50.

    $$LiquidationRemains= deposits + NPV_{account}-LiquidationProceeds$$

    - **Step 2: Parameter Values**

        Alice's NPV = $0
        Alice's NAV =  $5.50
        Carol NPV = $0
        Carol NAV = 20 + 4.50 = +$24.50

    Since the liquidator has to enter into a short position in order to close the long position, the longs and shorts will remain balanced along with their respective deposits and positions.