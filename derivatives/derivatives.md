# Injective Perpetuals Protocol

A decentralized BitMEX. Injective Protocol facilitates the creation and bilateral trading of perpetual swaps.  

# Overview

This protocol enables traders to create, enter into, and execute decentralized perpetual swap contracts on any arbitrary market. 

Our design adopts an account balance model where all positions are publicly recorded with their respective accounts. Architecturally, the core logic is implemented on on-chain smart contracts while the order matching is done through on the Injective Chain (in an off-chain relay on-chain settlement fashion). 

In our system, users have **accounts** which manage their **positions** in one or more futures **markets.** Each futures market specifies the bilateral futures contract parameters and terms, including most notably the margin ratio and oracle. Buyers and sellers of contracts in a futures market coordinate off-chain and then enter into contracts through an on-chain transaction. At all times, the payout (NPV) of long positions are balanced with corresponding short positions. The positions held by an account are subject to liquidation when the NAV of the account becomes negative. 

While a market is live, market-wide actions may affect all positions in the market such as dynamic funding rate adjustment (to balance market long/shorts) as well as clawbacks in rare scenarios. 

## **Key terms**

**Perpetual Swap** - A derivative providing exposure to the value of an underlying asset which mimics a margin-based spot market and has no expiry/settlement. 

**CFD** - Contract for difference which is a derivative stipulating that the sellers will pay the buyers the difference between the current value of an asset and its value at the contract's closing time.

**Address** - A secp256k1 based address. It's important to differentiate an address from an account in the futures protocol.

**Account** - An address can own one or more accounts which hold positions. Isolated margin positions are each held by their own account while cross margined positions are held by the same account.

**Deposits** - The value of an account's aggregate deposits denominated in a specified base currency (e.g. USDT). 

**Market** - A market is defined by its supported base currency, oracle, and minimum margin ratio / margin requirement. The market may also support a unique funding rates calculation, which is a distinctive feature of a perpetual swap futures market. 

**Direction** - Long or Short.

**Index Price** - The reference spot index price of the underlying asset that the derivative contract derives from. This value is obtained from an oracle. 

$$
P_{index}=\mathrm{index\ price\ of\ underlying}
$$

**Contract Price -** The value of the futures contract the position entered/created. This is purely market driven and is set by individuals, independent of the oracle price.


$$
P_{contract}=\mathrm{contract\ price}
$$
**Quantity** - The quantity of contracts. Must be a whole number. 
$$
quantity = \mathrm{quantity\ of\ contracts}
$$

**Position** - Executing a contract creates two opposing positions (one long and one short) which reflects the ownership conditions of some quantity of contracts.

A cross margined account can hold multiple positions while an isolated margined account holds one position.

**Margin** - Margin refers to the amount of base currency that a trader posts as collateral for a given position. 

**Contract** - A contract refers to the single unit of account for a position in a given direction in a given market. There are two sides of a contract (long or short) that one can enter into. 


**Order** - A cryptographically signed message expressing an unilateral agreement to enter into a position under certain specified terms (i.e. make order or take order). 

**Funding** - Funding refers to the periodic payments exchanged between the traders that are long or short of a perpetual contract at the end of every funding period (e.g. every 8 hours). When the funding rate is positive, longs pay shorts. When negative, shorts pay longs. 

**Funding Rate** - The funding rate value determines the funding fee that positions pay and is based on the difference between the price of the contract in the perpetual swap markets and spot markets. Further details can be found below. 

**Funding Fee** - The funding fee $$f$$ refers to the funding payment that is made for a **single contract** in the market for a given epoch $$i$$ and cumulative funding $$F$$ refers to the cumulative funding for the contract since position entry. 
$$
f_{i} = \mathrm{unit\ funding\ fee\ at\ epoch\ }i
$$

$$
F_{entry}= \sum \limits_{i=entry}^{current-1} f_i
$$

Hence, $$f_1=0.01$$ means that open long contracts at this time will pay 0.01 units of base currency and $$f_2 = -0.03$$ means that shorts should pay 0.03 units of base currency. 

**NPV** - Net Present Value of a single contract. For long and short perpetual contracts, the NPV calculation is:
$$
NPV_{perpetual\ long}= P_{index}-P_{contract} - F_{entry}
$$
$$
NPV_{perpetual\ short} = P_{contract}-P_{index}+F_{entry}
$$

**Liquidation -** When an account's **NAV** becomes negative, all of its positions are subject to liquidation, i.e. the forced closure of the position due to a minimum collateral requirement being breached. 

**Liquidation Penalty** - The liquidation penalty is a fixed percentage defining the value of the liquidated position that is paid out. Each market can define its own liquidation penalty (e.g. 5% in the example below) but every market's liquidation penalty must be greater than the global minimum liquidation penalty. 

$$
penalty = 0.05
$$

**Minimum Margin** - The minimum margin refers to the minimum amount of margin that a single contract must have at all times and is defined as:
$$
minMargin = P_{index} \cdot  penalty
$$

**Initial Margin** -  The initial minimum margin defines an additional component of the minimum margin requirement that every position in a given market must satisfy when first created. This requirement is stricter than the minimum margin requirement and exists in order to prevent unnecessary liquidation. 

Each market can define its own initial margin (e.g. 0.5 in the example below) but every market's margin ratio must be greater than the global minimum margin ratio. 
$$
initialMargin=penalty+initialMarginFactor=0.05
$$

Upon position creation, each long and short being matched must satisfy the following:

$$
\frac{margin_{long}}{quantity} \geq P_{contract}\cdot initialMargin
$$
$$
\frac{margin_{short}}{ quantity} \geq (2\cdot P_{index}-P_{contract})\cdot initialMargin
$$

By doing so the net margin deposited is invariant with respect to the **Contract Price** (which is determined by by the p2p market) and only changes with respect to the **Index Price**, satisfying the following:

$$
\frac{margin_{long}+margin_{short}}{quantity} \geq 2\cdot P_{index}\cdot initialMargin
$$

**NAV** - Net asset value of an account, important to differentiate from NPV. Used mainly for liquidation or dispute calculation. Follows the following formula:

$$
NAV = \sum_{positions} (margin + quantity\cdot (NPV - minMargin))
$$

margin +quantity *(Pindex- Pcontract - Pindex * penalty)

margin +quantity *(Pcontract-Pindex - Pindex * penalty)

**Liquidation Fee** - The fee that a liquidated position pays. The proceeds from liquidation fees are split between the liquidator and insurance fund.

$$
liquidationFee =penalty\cdot P_{index} \cdot quantity
$$

**Clawback** - A clawback event occurs when a threshold number of accounts cannot be liquidated with a net-zero final payout. In practice, this happens when a chain reaction of unidirectional liquidation cannot be liquidated with the position margin due to volatility or lack of liquidity. 

