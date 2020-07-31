# Examples

* **Perpetual Swap example**

  As an example, consider the mBTCUSDT perpetual swap contract. 1 contract represents the USDT value of exposure to 1 mBTC \(0.001 BTC\) which has an **Index Price** of 8 USDT when the spot price of Bitcoin is $8000.

  Two parties \(with Larry representing a Long, and the Sally representing a Short\) may enter into a contract with a given **Contract Price**, where the Long purchases synthetic Long exposure to mBTC and the Short purchases synthetic Short exposure to mBTC.

  Suppose Larry and Sally both decide to enter into a **Perpetual** contract with a **Funding Period** of 12 hours with a **Contract Price** of 8 USDT and each decide to use 8 USDT of their **Deposits** as **Margin** \(1x leverage, fully collateralized\).

  At the end of the funding period, suppose the **Funding Fee** is 0.01. Since the Funding Fee is positive, Longs must pay Shorts 0.01 USDT for every contract they own. Note: due to transaction costs associated with this model, the implementation of the funding is done differently and is discussed below but yields the same net result as described in this example.

* **CFD Example**

  As an example, consider the mBTCUSDT perpetual swap contract. 1 contract represents the USDT value of exposure to 1 mBTC \(0.001 BTC\) which has an **Index Price** of 8 USDT when the spot price of Bitcoin is $8000.

  Two parties \(with Larry representing a Long, and the Sally representing a Short\) may enter into a contract with a given **Contract Price**, where the Long purchases synthetic Long exposure to mBTC and the Short purchases synthetic Short exposure to mBTC.

  Suppose Larry and Sally both decide to enter into a **CFD** contract with a **Contract Price** of 8 USDT and each decide to use 8 USDT of their **Deposits** as collateral \(1x leverage, fully collateralized\).

  Unlike a Perpetual contract, a **CFD** has an expiration time when the contract is settled. At settlement, the Long will pay the Short the difference between the **Contract Price** and the **Index Price** of the underlying \(mBTC\) at expiration.

  Suppose that upon settlement, the Index Price of mBTC has risen to 10 USDT. Since **Contract Price - Index Price** = -2, the Short will pay the Long 2 USDT. Had the Index Price of mBTC been 6, the Long would have paid the Short 2 USDT.

  Perpetual swaps do not have an expiration time but rather are settled continuously with a **Funding Rate**, the details of which are specified below.

The above examples were for a single **Contract**. For multiple contracts, **Quantity** is used.

* **Example**

  \(A BIT INACCURATE SINCE THE SHORT IS UNREALISTIC UNLESS WE SUPPORT NEGATIVE PRICES WHICH WE DONT\) Suppose 1 mBTC contract represents 1/1000 of a BTC contract. Say BTC is 8000 USDT now. I want long exposure to 0.5 BTC, BUT I'm cheap and want to see if some idiot will give me exposure to 0.5 BTC \(500 mBTC\) at a rate of 10 USDT/BTC \(so 10 USDT/ 1000 mBTC which equals 0.01 USDT/mBTC\).

  Market details:

  * 1 contract = 1mBTC = 1/1000 BTC
  * Max leverage = 10x
  * liquidation penalty = 5%
  * minMargin = 5%

    `makerAssetAmount` = contract price of the 1mBTC = 0.01 USDT/mBTC

    `takerAssetAmount` = quantity of contracts

    `uint256 makerFee` - The amount of margin of base currency the maker would like to post/risk for the order.

    So I create an order where

  * makerAddress = cheap dude address
  * `makerAssetAmount` = 0.01 USDT/mBTC \(a 800x "discount" - a huge bargain!\)
  * `takerAssetAmount` = 500 mBTC
  * `makerFee` = 0.5 USDT

    When a relayer receives this order, the relayer should check whether the account is sufficiently collateralized. He can check by looking at the account's margin.

  * contract price  _quantity = 0.01_  500 = $5
  * Cheap guy USDT margin in "my account ID"= $0.50
  * min collateral ratio = minMargin + liquidation penalty = 10%
  * The cheap guy can have any amount of margin in his account, so long as it satisfies the min collateral ratio of the contract value, aka 0.1\*5 = $0.50

    $$deposit \geq quantity \cdot P_{contract} \cdot (marginRatio + penalty)$$

    $$deposit \geq 500 \cdot 0.01 \cdot (0.05+0.05) = 0.5$$

    $$deposit =order.makerFee \geq 0.5$$

  * Cheap guy collateral ratio = margin / contract value = $0.5 / $5 = 10%
  * FYI, the relayer should consistently check the collateralization of cheap guy's make order long as its on the orderbook. Because if the USDT value or other margin he's using \(INJ for example\) has a sudden drop, the position might be no longer sufficiently collateralized.

    The Oracle provides the index price, aka the value denominated in baseCurrency of 1 mBTC contract.

  * `getPrice` returns 8 USDT

    $$P_{index}=8$$

    $$NPV_{perpetual\ long}= quantity \cdot (P_{index}-P_{contract} - (F_{current} - F_{entry}))$$

    $$NPV_{account}=NPV_{perpetual\ long}= 500 \cdot (8-0.01 - (0 - 0)) = 3995$$

    In this example the Account NPV equals the NPV of the perpetual long since the maker only has one position.

    NPV \(account\) assume if filled = $8 - $0.01 = $7.99

    $$NAV =Deposits +NPV_{account}- \sum_{positions} (minMargin)$$

    $$NAV = 0.5 + 3995 - 8\cdot 500\cdot (0.05 + 0.05) = 3595.5$$

    NAV = $7.24

    Contract Price calculation \(basically assume that by the time the contract is executed, the oracle price = contract price\):

    Contract price= $0.01

    NAV= $0.5 + 0 - \(10%\)  _$0.01_  500

    NAV= 0

    The futures protocol goes by the oracle price calculation, but the relayer should calculate both. Because the relayer should operate under the assumption that by the time the order is taken, the oracle price is approximately the same as contract price. So both NAV calculations have to be above or = 0.

    The cheap guy's position is sufficiently collateralized, therefore the relayer accepts the cheap guy's position.

    For the idiot taker, we must have

    0 ≤ NAV = Deposits + NPV\_account - sum\(minMargins\)

    -1_Deposits ≤ quantity_  \(P\_contract - P\_index\) - \(P\_index  _quantity\)_\(marginRatio + penalty\)

    Deposits ≥ -1_quantity_  \[\(P\_contract - P\_index\) - \(P\_index\)\*\(marginRatio + penalty\)\]

    Deposits ≥ quantity  _\[P\_index - P\_contract + P\_index_\(marginRatio + penalty\)\]

    $$deposit \geq quantity \cdot (P_{index}-P_{contract}+P_{index}\cdot(marginRatio + penalty))$$

    $$deposit \geq 500 \cdot (8-0.01+8\cdot(0.05+0.05)) = 4395$$

    $$deposit +quantity \cdot P_{contract} \geq quantity \cdot P_{index}+quantity*P_{index}\cdot(marginRatio + penalty)$$

    **Make Order 2 \("Take Order" from idiot\)**

    This denotes a make order for short exposure to 0.5 BTC \($4000 value\). This make order says I will give makerAssetAmount = ~3995 USDT or whatever the lowest amount so that the NAV for the position of $10 at $8000 price oracle to be 0 to get short exposure on 0.5 BTC at $10. In practice,

    Selling 1 mBTC for 0.01 baseCurrency

  * `makerAssetAmount` = 0.01 USDT/mBTC
  * `takerAssetAmount` = 500
  * `makerFee` ≥ ???

    Once the idiot provides the fillOrder, the relayer can check collateralization on the idiot or send it straight to futures protocol. This depends on relayer's responsibility

    How much collateral the cheap guy needs:

    Oracle Price calculation:

    $$NPV_{perpetual\ short} = quantity \cdot (P_{contract}-P_{index}+(F_{current}-F_{entry}))$$

    $$NPV_{perpetual\ short} = 500 \cdot (0.01-8+(0-0)) =-3995$$

    NPV \(account\) assume if filled = -3995

    0 = margin deposit - $7.99_500 - \(10%\)_$8\*500

    Margin deposit = 3995+400

    = $4395P\_{contract}

    $$NAV =Deposits +NPV_{account}- \sum_{positions} (minMargin)$$

    $$NAV =Deposits -3995- 8\cdot 500\cdot (0.05+0.05)$$

    NAV = 0

    In order to execute successfully, both parties' NAVs must always be greater than 0. This means that the idiot must have at least -1_\(-3995-8_500\*\(0.05+0.05\) = 4395 USDT in his deposits to use as margin.

    Contract Price calculation \(basically assume that by the time the contract is executed, the oracle price=contract price\):

    Contract price= $0.01

    NAV= $0.5 + 0 - \(10%\)  _$0.01_  500

    NAV= 0

    In order for both positions to be sufficiently collateralized, the idiot needs to post $4395 as margin\(lol\). It's worse than 1x leverage because the idiot has to also post collateral for margin deposit. So he's effectively over collateralizing his position.

[On Closing Positions and Liquidations](https://www.notion.so/On-Closing-Positions-and-Liquidations-f92ff360a0f54576bfe1e55a13d42dbf)

[Smart Contract Implementation](https://www.notion.so/Smart-Contract-Implementation-3130ec499e5748bba65962ad9d3a1eb7)

## Basic Futures Walkthrough

OUTDATED, NEED TO UPDATE

* Formulas Reference

  For reference, the fomulas used are provided below:

  $$minMargin = (marginRatio + penalty) \cdot P_{index} \cdot quantity$$

  $$NPV_{long\ position}= - NPV_{short\ position}=quantity \cdot (P_{index}-P_{contract})$$

  $$NPV_{account}=\sum_{positions}{NPV_{position}}$$

  $$NAV =deposit +NPV_{account}-\sum_{positions} (minMargin)$$

* Simple Scenario Walkthrough

  For simplicity, we denote values in USD as a proxy for `baseCurrency`. For cases when the INJ token is used as the base currency, denoting the values with dollar signs \(e.g. $20\) should be replaced with INJ \(e.g. 20 INJ\). We also consider Alice and Bob's position NPVs as their accounts NPVs since Alice and Bob each only have 1 position.

  Suppose the price of 1 ETH = $100.

  1. Market for fETH \(future ETH\) created with: 
     * Margin ratio of 10%
     * Liquidation penalty of 5%
     * Expiration date in 1 month
     * Contract Price of $100 \(i.e. 1 fETH contract represents the value of 1 ETH\)
  2. Alice and Bob both deposit $20 of `baseCurrency` into the Accounts contract.
  3. **Parameter Values**

     Alice's NAV = +$20 Bob's NAV = +$20

  4. Alice creates a signed Order Message to enter 1 long position for a fETH and sends this to the Injective Sidechain.
  5. Bob sees Alice's order and decides to fill it, establishing his short position and Alice's long position.
  6. **Parameter Values**

     Alice's NPV = quantity_\(Pindex - Pcontract\) = 1_\(100 - 100\) = $0 Alice's NAV = deposits + NPV - minMargin = 20 - 0 - \(0.10 + 0.05\)_100_1 = +$5 Bob's NPV = quantity_\(Pcontract - Pindex\) = 1_\(100 - 100\) = $0 Bob's NAV = deposits + NPV - minMargin = 20 - 0 - \(0.10 + 0.05\)_100_1 = +$5

  7. The price of ETH drops to $90.
  8. **Parameter Values**

     Alice's NPV = 1_\(90 - 100\) = -$10 Alice's NAV = 20 - 10 - \(0.10 + 0.05\)_90_1 = -$3.50 Bob's NPV = -1_\(100 - 110\) = +$10 Bob's NAV = 20 + 10 - \(0.10 + 0.05\)_90_1 = +$16.50

     Alice's account is now subject to liquidation as her NAV is negative.

  9. To liquidate Alice, Carol a third-party liquidator opens a counterbalancing short position of 1 fETH to net out her long position, which first requires Carol to find a long order from the orderbook and fill it. Let Carol also have $20 in her account deposits.
  10. **Step 1: Parameter Values**

      Alice's NPV = 1_\(90 - 100\) = -$10 Alice's NAV = 20 - 10 - \(0.10 + 0.05\)_90_1 = -$3.50 Carol NAV = 20 + 0 - \(0.10 + 0.05\)_90 = +$6.50

      In the liquidation, Alice will have the following amount refunded to her

      The liquidator will earn the following liquidation proceeds which equals 0.05_90_1 = +$4.50

      $$LiquidationProceeds= penalty\cdot P_{index}*quantity$$

      while Alice has the following liquidation remains to her which equals 20 - 10 - 0.05_90_1 = $5.50.

      $$LiquidationRemains= deposits + NPV_{account}-LiquidationProceeds$$

  11. **Step 2: Parameter Values**

      Alice's NPV = $0 Alice's NAV = $5.50 Carol NPV = $0 Carol NAV = 20 + 4.50 = +$24.50

      Since the liquidator has to enter into a short position in order to close the long position, the longs and shorts will remain balanced along with their respective deposits and positions.

