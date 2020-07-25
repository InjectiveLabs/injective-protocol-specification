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