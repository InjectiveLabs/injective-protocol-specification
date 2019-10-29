# Injective Protocol Token Economics

## Introduction

We describe Injective Token (INJ) and its use case in Injective Protocol. INJ will used for the following purposes:

1.  [Stake-Based Fee Distribution](#stake-based fee distribution)
    1.  [Decentralized Buy-Back Auction](#decentralized-buy-back-auction) 
1.  [Market Maker Incentives](#market-maker-incentives)
1.  [Relayer Incentives](#relayer-incentives)
1.  [Sidechain Governance](#sidechain-governance)
1.  [Proof of Stake Security](#proof-of-stake-security)

We show the utility of the INJ token in the aforementioned use cases and demonstrate how they result in value accrual of our token. 

## Stake-Based Fee Distribution
Exchange fees collected from all trading pairs are aggregated monthly into a "basket of tokens" and sold as a batch to the public which can bid on the basket with INJ tokens. To achieve this, we utilize a simple public auction mechanism that repeats every 1 month equivalent of blocks. The Injective coordinator contract aggregates all of the exchange fees collected over the 1 month period and then conducts a two-day public auction. 

During the auction, one can bid on the auction by calling the auction contract `bid` function which transfers the bidder's INJ token amount to the auction contract (which of course can only occur if the bidder has set the necessary ERC-20 `approval`). At any given point in time, only the current highest bidder will have their INJ tokens locked in the auction contract, thus allowing any bidders who have been outbid to withdraw their funds whenever they so desire. After the auction period elapses, the winner can trigger the disbursement of the accrued funds to their account. 



