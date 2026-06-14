# Learn: Understanding SatoshiNet

The Learn section is for readers who are new to SAT20 and SatoshiNet. It does not assume prior knowledge of STP, RSMC, UTXO, or smart contract internals. It starts with the core concepts.

SatoshiNet aims to become a Bitcoin-native extension network. It does not serve a single asset only. It serves Bitcoin L1 native assets: BTC, Ordinals, Runes, BRC20, ORDX, and future UTXO-based asset protocols.

## Reading Order

1. [Why Bitcoin Needs a Native Extension Network](bitcoin-native.md)
2. [Asset Safety Model](security-model.md)
3. [Indexer: Bitcoin Asset Fact Layer](indexer.md)
4. [STP Introduction](stp.md)
5. [Smart Contracts and GAS](smart-contracts-and-gas.md)
6. [AI Agents and User Asset Control](ai-agent.md)

## Core Concepts

| Concept | Summary |
| --- | --- |
| Bitcoin-native extension network | Assets originate from Bitcoin L1, and the exit and safety boundary can return to Bitcoin L1 |
| Indexer | Expresses asset states from different Bitcoin L1 protocols as queryable and verifiable asset facts |
| STP | Protocol for moving assets into, out of, and back under channel protection |
| Commitment transaction | A pre-signed transaction a user can use to exit in abnormal situations |
| Punishment transaction | A transaction used to protect user assets when the peer broadcasts an old state |
| Smart contract | Application logic running on SatoshiNet |
| GAS | Asset used to pay for contract execution and network resource consumption |
| AI Agent | An automated assistant that helps users understand, verify, and execute complex on-chain operations |

## What You Should Understand

After reading Learn, readers should be able to answer:

1. Why SatoshiNet is not a normal custodial bridge.
2. Why the indexer is the asset fact layer for STP, wallets, exchanges, and Agents.
3. How STP lets users retain asset control.
4. Why smart contracts expand SatoshiNet's application boundary.
5. Why GAS is an important economic entry point for the ecosystem.
6. Why AI Agents are well suited for STP and wallet operations.
