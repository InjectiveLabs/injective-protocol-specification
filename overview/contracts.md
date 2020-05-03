# Injective Contracts

Injective Protocol is token-based protocol which is inextricably tied to the INJ token \(an ERC-20 token\). As such, key components of protocol interactions and token economics are implemented through the following smart contracts: 

**Injective Coordinator Contract**   
The Injective's Coordinator Contract services both 0x-based orders as well as Injective's derivative transactions on Ethereum as well as on the Injective Chain. The principal purpose of the coordinator is to serve as a liquidity solution enabling more competitive pricing by preventing front-running and allowing for much lower latency trading. 

The Injective Coordinator Contract follows the [0x v2 Coordinator](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/coordinator-specification.md) specification for both normal 0x transactions and also Injective derivatives transactions. However, unlike traditional implementations which only require one signature from a centralized coordinator, Injective's decentralized coordinator enforces transactions to have a minimum threshold of coordinator signatures in order for a transaction to be approved. These required signatures are provided through the application logic built into the consensus of the Injective Chain. Each coordinator is bonded through INJ stake and can be slashed if they "go rogue" and independently provide signatures for non-sidechain approved transactions. 

Current deployments of the Injective Coordinator Contract can be found here:

| Network                | Contract Address       |
| ------------------- | ------------------- |
|  Kovan          |     `0x30493852999f5091d2430B6a1222Aa816237a486`      |


**Staking Contract**  
The Injective Staking Contract manages the core functions for stakers in Injective Protocol including slashing, rewards, delegation and governance. 

**Injective Futures Contracts**  
The Injective Futures Protocol encompasses a suite of smart contracts. Comprehensive details can be found [here](https://github.com/InjectiveLabs/injective-futures). 

**Injective Bridge Contracts**  
The Injective Bridge Contracts encompass a suite of smart contracts managing the two-way peg between Ethereum and the Injective Chain. More details can be found [here](https://github.com/InjectiveLabs/injective-core). 

**Injective Token Contract**  
The Injective Token Contract is an ERC-20 contract for the INJ token.
