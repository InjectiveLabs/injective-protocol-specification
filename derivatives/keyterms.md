# Key Terms

## **Perpetual Swap**

A derivative providing exposure to the value of an underlying asset which mimics a margin-based spot market and has no expiry/settlement.

## **CFD**

Contract for difference which is a derivative stipulating that the sellers will pay the buyers the difference between the current value of an asset and its value at the contract's closing time.

## **Address**

A secp256k1 based address. It's important to differentiate an address from an account in the futures protocol.

## **Account**

An address can own one or more accounts which hold positions. Isolated margin positions are each held by their own account while cross margined positions are held by the same account.

## **Deposits**

The value of an account's aggregate deposits denominated in a specified base currency \(e.g. USDT\).

## **Market**

A market is defined by its supported base currency, oracle, and minimum margin ratio / margin requirement. The market may also support a unique funding rates calculation, which is a distinctive feature of a perpetual swap futures market.

## **Direction**

Long or Short.

## **Index Price**

The reference spot index price of the underlying asset that the derivative contract derives from. This value is obtained from an oracle.

$$
P_{index}=\mathrm{index\ price\ of\ underlying}
$$

## **Contract Price**

The value of the futures contract the position entered/created. This is purely market driven and is set by individuals, independent of the oracle price.

$$
P_{contract}=\mathrm{contract\ price}
$$

## **Quantity**

The quantity of contracts. Must be a whole number.

$$
quantity = \mathrm{quantity\ of\ contracts}
$$

## **Position**

Executing a contract creates two opposing positions \(one long and one short\) which reflects the ownership conditions of some quantity of contracts.

A cross margined account can hold multiple positions while an isolated margined account holds one position.

## **Margin**

Margin refers to the amount of base currency that a trader posts as collateral for a given position.

## **Contract**

A contract refers to the single unit of account for a position in a given direction in a given market. There are two sides of a contract \(long or short\) that one can enter into.

## **Order**

A cryptographically signed message expressing an unilateral agreement to enter into a position under certain specified terms \(i.e. make order or take order\).

## **Funding**

Funding refers to the periodic payments exchanged between the traders that are long or short of a perpetual contract at the end of every funding epoch \(e.g. every 8 hours\). When the funding rate is positive, longs pay shorts. When negative, shorts pay longs.

## **Funding Rate**

The funding rate value determines the funding fee that positions pay and is based on the difference between the price of the contract in the perpetual swap markets and spot markets.

## **Funding Fee**

The funding fee $$f$$ refers to the funding payment that is made for a **single contract** in the market for a given epoch $$i$$.

$$
f_{i} = \mathrm{unit\ funding\ fee\ at\ epoch\ }i
$$

Cumulative funding $$F$$ refers to the cumulative funding for the contract since position entry.

$$
F_{entry}= \sum \limits_{i=entry}^{current-1} f_i
$$

## **NPV**

Net Present Value of a single contract. For long and short perpetual contracts, the NPV calculation is:

$$
NPV_{perpetual\ long}= P_{index}-P_{contract} - F_{entry}
$$

$$
NPV_{perpetual\ short} = P_{contract}-P_{index}+F_{entry}
$$

## **Liquidation**

When an account's **NAV** becomes negative, all of its positions are subject to liquidation, i.e. the forced closure of the position due to a minimum collateral requirement being breached.

## **Liquidation Penalty**

The liquidation penalty is a fixed percentage defining the value of the liquidated position that is paid out. Each market can define its own liquidation penalty \(e.g. 3% in the example below\) but every market's liquidation penalty must be greater than the global minimum liquidation penalty.

$$
penalty = 0.03
$$

## **Initial Margin Requirement**

When a position is first created, the amount of collateral supplied as margin must satisfy the initial margin requirement. This margin requirement is stricter than the maintenance margin requirement and exists in order to reduce the risk of immediate liquidation.

$$
initialMarginRatio=penalty+initialMarginRatioFactor
$$

Upon position creation, each contract must satisfy the following margin requirement:

$$
\frac{margin}{quantity} \geq \max (P_{contract} \cdot initialMarginRatio, P_{index} \cdot initialMarginRatio - NPV)
$$

## **Maintenance Margin Requirement**

The maintenance margin requirement refers to the minimum amount of margin that a position must maintain after being established. If this requirement is breached, the position is subject to liquidation.

Throughout the lifetime of a position, each contract must satisfy the following margin requirement:

$$
\frac{margin}{quantity} \geq P_{index} \cdot penalty- NPV
$$

## **Clawback**

A clawback event occurs when a threshold number of accounts cannot be liquidated with a net-zero final payout. In practice, this happens when a chain reaction of unidirectional liquidation cannot be liquidated with the position margin due to volatility or lack of liquidity.

