# Governance

Governance occurs on two separate portions of our protocol: the sidechain application and the coordinator smart contract.

## Sidechain Governance

The sidechain governance is built on top of the core Tendermint consensus. Validators for the Tendermint consensus process are incentivized by block reward and punished by slashing if a malicious behaviors were detected. This process is elaborated in [Validator Requirements](governance.md#validator-requirements).

## Coordinator Contract Governance

The coordinator contract only approves blocks of trade that has accumulated enough signature from a supermajority of validators from the sidechain. The coordinator contract maintains a list of validators that can be updated from the sidechain if there are supermajority agreement from the previous list of validators.

Once the trades are approved, the coordinator contract can submit the trades to 0x for settlement.

### Voting mechanism

Injective's native token holder can participate in governing the coordinator contract on Ethereum. They have the power to vote on key decisions such as protocol upgrade, listing, fee schedule, and modifying other key variables inThe protocol's [native token](governance.md#token-economics) will be used to maintain proof-of-stake security on the sidechain, reward order discovery and origination for nodes, and allow token holders to capture value on the success of the protocol via a token burn or distribution mechanism. The exchange protocol does not collect fees in native token by default but rather implements a negative spread model like most of the traditional centralized exchanges. The fees collected will undergo a periodic auction enforced on a smart contract to buy back the native token. the exchange. Our protocol allows token holders to create proposals that can be voted on by the community with their tokens.

### Creating a Proposal

In order to create a proposal for all token holders to vote on, a proposer must lockup more than the `Minimum_proposal_requirement` to open a voting period.

After a proposal is successfully created, a pending period begins where the proposal must accumulate enough lockup to surpass the `Minimum_referendum_requirement` before `Proposal_pending_period` \(denominated in blocks\) expires. The `Minimum_referendum_requirement` will be calculated as a percentage of the circulating supply of Injective's native token. If the proposal surpasses the requirement before the pending period ends, it will successfully become a referendum that the general token holders can vote on.

Once then pending period ends, the lockup can be withdrew regardless of the outcome. Depending on the proposal's subject matter, the variables `Minimum_referendum_requirement` and `Proposal_pending_period` may be different.

### Voting on a Proposal

Once a proposal becomes a referendum, a `Referendum_period` will begin. During the period, token holders can vote by creating a `Vote` transaction. By doing so, the token holder is signaling yay or nay on the proposal with the voting power proportional to their wallet balance. Once a vote is finalized, anyone can call the smart contract to tally the vote and reach a decision. However, if the final yay vote does not pass the `Minimum_yes_requirement`, the referendum is considered fail due to the lack of participation. Like `Minimum_referendum_requirement`, the `Minimum_yes_requirement` is based on a percentage of the token's circulating supply.

If a proposal is successfully voted into implementation, then the smart contract will enforce the decision deterministically.

