# Oracle

## General concept

The oracle serves two functions:

1. Update the index price of an asset
2. Set the new funding rate

### 1. Update the index price

The index price is used to calculate the NPV of positions. It will be periodically updated by the oracle.

### 2. Set the new funding rate

The funding rate is a critical piece to ensure convergence of market prices to the real underlying asset price. It consists of two components:

1. Premium Index: The premium index measures the difference between the perpetual swap’s price and the underlying asset it’s tracking.
2. Interest Rate: The interest rate is a function of interest rates between base and quote currency where each currency has its own defined rate.

The exact formula are:

`Premium = (max(0, Impact Bid Price — Index Price) — max(0, Index Price — Impact Ask Price)) / Index Price`

`Funding Rate = Premium + clamp(Interest Rate — Premium, 0.05%, -0.05%)`

The impact prices are weighted averages over the first 3000 long or respective short orders.

## Testnet setup

For the testnet we are setting up a centralized oracle service. In later versions this will be replaced by a decentralized mechanism.

### Testnet config

* Pair = XAU/USDT
* Gold Interest Rate = 0.03%
* USD Interest Rate = 0.06%
* Funding Interval = 8 hours

### Oracle service

The oracle service updates the ticker prices every 5 minutes. If any asynchronous requests fail, we deploy an [exponential backoff](https://cloud.google.com/iot/docs/how-tos/exponential-backoff) strategy. Prices are taken from [https://metals-api.com/](https://metals-api.com/).

Three times per day the funding rate is calculated according to the formula from above.

