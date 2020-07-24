# Token Economics

The full specifications for Injective Protocol's token economics can be found [here](https://github.com/InjectiveLabs/injective-protocol-specification/blob/master/token-economics.md).

Injective Protocol's native token is used for the following purposes:

1. Proof-of-stake reward

   Validators can stake with the token and receive block reward proportional to stake.

2. Buy-back and Burn

   Exchange fees collected via negative spread model are periodically auctioned off through smart contract to buy back Injective's native token for reward distribution. The proceeds from the auction will be burned.

3. Node incentive model

   Hosting a node and participating in Relayerd will receive a percentage of fees generated from the orders they discover. This will incentivize nodes to submit VDF time proof on the user's behalf and create a referral system where the user and the node can receive reward and trading fee discounts.

We will implement these features outside of the core protocol.

1. Market Maker Incentives

   Make orders will receive a net positive fee rebate to incentivize liquidity. Distribution will happen periodically based on snapshots.

2. PNL incentive

   For a limited period of time, a fixed number of tokens will be periodically distributed to users weighted by their net profit and loss during that time period. The distribution will end when a predetermined cumulative number of tokens is distributed.

Future implementation:

1. Governance

   Reference [Coordinator Contract Governance](token-economics.md#coordinator-contract-governance)

