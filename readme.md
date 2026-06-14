# SAT20 and SatoshiNet

SAT20 is a protocol and network stack built around Bitcoin-native assets. Its core goal is to let BTC, Ordinals, Runes, BRC20, ORDX, and other Bitcoin L1 assets move into a lower-cost, faster, programmable environment while preserving user control.

SatoshiNet is the Bitcoin-native extension network in the SAT20 ecosystem. It is not a custodial bridge, and it is not an independent asset system unrelated to Bitcoin. Assets enter SatoshiNet from Bitcoin L1; asset facts are expressed by indexers; the safety boundary is formed by STP channels, commitment transactions, punishment transactions, force-close paths, and verifiable Bitcoin L1 transactions.

## Core Thesis

The Bitcoin ecosystem has long lacked a network that satisfies three requirements at the same time:

1. Assets originate from Bitcoin L1, instead of being custodially bridged or created from nothing.
2. Users retain an exit path, even when a Core Node fails or behaves maliciously.
3. The network is cheap, fast, and programmable enough for trading, contracts, AI Agents, and richer applications.

SatoshiNet is built to solve this problem.

## Four Foundations

| Foundation | Role |
| --- | --- |
| STP | Moves Bitcoin L1 assets into, across, and out of SatoshiNet while preserving user control |
| Indexer | Expresses BTC, Ordinals, Runes, BRC20, ORDX, and other L1 assets as queryable, verifiable asset facts |
| Smart Contracts | Turn SatoshiNet from an asset circulation layer into an application network |
| GAS | Provides the economic entry point for contract execution, transaction processing, and ecosystem incentives |

STP and indexers are the foundation for moving assets into SatoshiNet. The indexer tells wallets which assets exist on Bitcoin L1, which UTXOs carry them, and whether their state is confirmed. STP brings those assets into a channel safety boundary where users can exit and punish revoked states. Smart contracts and GAS then allow those assets to participate in trading, market making, payments, contracts, and more complex applications.

AI Agents are a new user interface. STP channel safety, commitment transactions, punishment transactions, and cross-layer states are complex for most users, but an Agent can read the evidence, run checks, call wallet adapters, and explain whether the next action is safe.

## Where to Start

If this is your first time learning about SAT20:

1. Read [Why Bitcoin Needs a Native Extension Network](learn/bitcoin-native.md).
2. Read [Asset Safety Model](learn/security-model.md).
3. Read [Indexer: Bitcoin Asset Fact Layer](learn/indexer.md).
4. Read [STP Introduction](learn/stp.md).
5. Read [Smart Contracts and GAS](learn/smart-contracts-and-gas.md).
6. Read [AI Agents and User Asset Control](learn/ai-agent.md).

If you want to build applications:

1. Start with the [Developer Center](build/readme.md).
2. Read the [Developer Quickstart](build/quickstart.md).
3. Choose the STP, Indexer, smart contract, or PWA adapter integration path.

If you want an AI Agent to operate STP:

1. Read the [Bitcoin Ecosystem AI Agent Asset Safety Standard](ai/bitcoin-agent-safety-standard.md).
2. Read [SAT20 Agent Wallet: Install and Use](ai/sat20-agent-wallet/readme.md).
3. Install the skill:

```bash
curl -fsSL https://raw.githubusercontent.com/sat20-labs/docs/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/scripts/install.sh | bash
```

4. Read the [SAT20 Agent Wallet Verification Matrix and Data Gaps](ai/sat20-agent-wallet/verification-and-data-gaps.md).

## Official Links

- Documentation: [docs.sat20.org](https://docs.sat20.org)
- Website: [sat20.org](https://sat20.org)
- X: [SAT20Labs](https://x.com/SAT20Labs)
- GitHub: [sat20-labs](https://github.com/sat20-labs)

## Documentation Structure

| Section | Content |
| --- | --- |
| Learn | Explains SatoshiNet, STP, Indexer, the safety model, smart contracts, GAS, and AI Agents |
| Use | Guides users through wallets, cross-layer movement, asset verification, and risk boundaries |
| Build | For developers, wallets, exchanges, and infrastructure teams |
| Protocol | For implementers and auditors: STP, SatoshiNet, ORDX, Indexer, and contract protocols |
| AI Agent | For Agent developers and wallet adapters that need verifiable asset control |
| Ecosystem | For builders, asset communities, exchanges, institutions, and community members |
| Roadmap | Describes ecosystem phases, public milestones, and the long-term direction |

This documentation will keep evolving. The priority is accurate protocol facts, reproducible testnet evidence, and executable developer paths. English docs, website content, and community material will continue to improve alongside the protocol.
