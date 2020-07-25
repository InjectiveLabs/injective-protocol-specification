- **Filling an Order**
    1. Taker discovers the make order from the Injective Chain and decides to fill some quantity of the order. This means that he wishes to enter into a position in the opposite direction of the make order at the Contract Price specified in the make order for some desired quantity of contracts using his desired amount of margin as collateral.
    2. Taker calls  `fillOrder(order, quantity, margin, signature)` on the futures smart contract. This will immediately enter the workflow of establishing two positions - one for the maker and one for the taker. 
    3. By doing so, the futures contract transfers `margin` amount of the base currency specified by the make order and take order using ERC-20 the `transferFrom` call and deposits the tokens into their balance. 
    4. The positions are attempted to be created for both parties. If the following conditions succeed, then the positions are created. If any one of these checks fail the entire transaction is reverted.

        $$\frac{margin_{long}}{quantity} \geq P_{contract}\cdot initialMargin$$

        $$\frac{margin_{short}}{ quantity} \geq (2\cdot P_{index}-P_{contract})\cdot initialMargin$$

        - If any of the orders being used was filled before, the `quantity` being taken must not be greater than what is remaining.

    Now, suppose a maker creates a huge make order and a taker decides to partial fill it. If it is the first time that the maker and taker are trading, by default, the entire maker's deposits are used as margin for this position, and the same is true for the taker. 