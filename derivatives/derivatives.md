# Injective Perpetuals Protocol

A decentralized BitMEX. Injective Protocol facilitates the creation and bilateral trading of perpetual swaps.  

# Overview

This protocol enables traders to create, enter into, and execute decentralized perpetual swap contracts on any arbitrary market. 

Our design adopts an account balance model where all positions are publicly recorded with their respective accounts. Architecturally, the core logic is implemented on on-chain smart contracts while the order matching is done through on the Injective Chain (in an off-chain relay on-chain settlement fashion). 

In our system, users have **accounts** which manage their **positions** in one or more futures **markets.** Each futures market specifies the bilateral futures contract parameters and terms, including most notably the margin ratio and oracle. Buyers and sellers of contracts in a futures market coordinate off-chain and then enter into contracts through an on-chain transaction. At all times, the payout (NPV) of long positions are balanced with corresponding short positions. The positions held by an account are subject to liquidation when the NAV of the account becomes negative. 

While a market is live, market-wide actions may affect all positions in the market such as dynamic funding rate adjustment (to balance market long/shorts) as well as clawbacks in rare scenarios. 

# Architecture

The Injective Futures Protocol uses off-chain order relay with on-chain settlement. In this approach, cryptographically signed orders are broadcast to the decentralized orderbook on the Injective Chain. Then, an interested counterparty may inject one or more of these orders into the Injective Futures Protocol contract to execute and settle trades directly to the blockchain. 

