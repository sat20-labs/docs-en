# Overview

The AI Agent section focuses on one question: how Agents can help users safely operate Bitcoin L1 and SatoshiNet assets without touching private keys or bypassing authorization, and gradually help BTC communities plan and deploy their own infrastructure.

SAT20's view is that the more complex the protocol, the more users need Agents to understand and verify it. STP commitment transactions, punishment transactions, cross-layer state, and abnormal recovery are evidence that Agents can read and reason about.

## Two Product Lines

| Product Line            | Goal                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------- |
| Agent Wallet & Safety   | Help users verify and operate assets inside the wallet safety boundary                                   |
| Community Builder Agent | Help BTC communities plan DEX, DAO, wallet, Indexer, Explorer, and contract modules through conversation |

## Entries

1. [Bitcoin Ecosystem AI Agent Asset Safety Standard](bitcoin-agent-safety-standard.md)
2. [AI Agents and User Asset Control](../learn/ai-agent.md)
3. [SAT20 Agent Wallet: Install and Use](sat20-agent-wallet/)
4. [SAT20 Agent Wallet Interoperability Skill Specification](sat20-agent-wallet/interoperability.md)
5. [SAT20 Agent Wallet Asset Safety Control Guide](sat20-agent-wallet/asset-safety.md)
6. [SAT20 Agent Wallet Verification Matrix and Data Gaps](sat20-agent-wallet/verification-and-data-gaps.md)
7. [SAT20 Agent Wallet Testnet Validation Record](sat20-agent-wallet/testnet-validation.md)
8. [Community Builder Agent](community-builder-agent.md)

## Principles

1. Agents do not store private keys.
2. Agents do not store seed phrases.
3. Agents do not bypass wallet authorization.
4. Agents do not rely only on balances.
5. Agents must check commitment transactions and punishment coverage.
6. Agents must stop value movement when safety evidence is missing.
