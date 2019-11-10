## Token Economics
The full specifications for Injective Protocol's token economics can be found [here](https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/token-economics.md). 

Injective Protocol's native token is used for the following purposes:

1. Proof-of-stake reward
Validators can stake with the token and receive block reward proportional to stake.
2. Fee Distribution
Exchange fees collected via negative spread model are periodically auctioned off through smart contract to buy back Injective's native token for reward distribution. The tokens are first distributed to relayers proportional by orders discovered/originated, and then to validators proportional by stake. Part of the tokens may also be burned instead of distributing to validators depending on final implementation. 


We will implement these features in future iterations.

3. Market Maker Incentives
Make orders will receive a net positive fee rebate to incentivize liquidity.
4. Fee Discount
Token holders may receive discount on exchange fees.
5. Governance
Reference [Coordinator Contract Governance](#coordinator-contract-governance)
