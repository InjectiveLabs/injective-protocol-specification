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