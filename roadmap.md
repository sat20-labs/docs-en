# Roadmap

The SAT20 / SatoshiNet roadmap is organized around one long-term goal: building a Bitcoin-native extension network where Bitcoin L1 assets can enter a programmable, low-cost, automatable application environment while preserving user control.

The roadmap is not a price promise and not a fixed-date schedule. It records current priorities, phase goals, and public milestones that need verifiable evidence.

## Phase 1: STP + Indexer Asset Foundation

Goal: prove that assets entering SatoshiNet are not entering a custodial bridge, but a cross-layer asset system supported by unified asset indexing and user-verifiable channels. The indexer expresses asset facts on Bitcoin L1; STP brings those assets into a channel safety boundary with exit capability and punishment for revoked states.

Focus:

1. Unified indexing of multi-protocol Bitcoin L1 assets.
2. L1/L2 ascend, descend, and channel asset evidence chains.
3. STP channel lifecycle.
4. Splicing-in / splicing-out.
5. Unlock / lock / lock-with-expand.
6. Commitment export.
7. Punish coverage.
8. Testnet old-commitment broadcast and punishment drills.
9. SAT20 Agent Wallet skill and PWA adapter.

Current docs:

- [Indexer: Bitcoin Asset Fact Layer](learn/indexer.md)
- [Indexer Integration and Asset Fact Layer](build/indexer.md)
- [STP Technical Whitepaper](protocol/stp/readme.md)
- [SAT20 Agent Wallet Asset Safety Control Guide](ai/sat20-agent-wallet/asset-safety.md)
- [SAT20 Agent Wallet: Install and Use](ai/sat20-agent-wallet/readme.md)

## Phase 2: Smart Contracts and GAS

Goal: evolve SatoshiNet from an asset circulation network into an application network.

Focus:

1. Template contracts.
2. AMM, limit orders, stablecoins, payments, and other basic applications.
3. GAS fee model.
4. Contract indexer and L2 state indexing.
5. Contract developer tooling.
6. EVM compatibility path.
7. Natural language contracts and AI Agent contract experiments.

Current docs:

- [Smart Contracts and GAS](learn/smart-contracts-and-gas.md)
- [Smart Contract Protocol](protocol/contracts/readme.md)
- [GAS Ecosystem Opportunity](ecosystem/gas.md)

## Phase 3: Developers and Infrastructure

Goal: lower the integration cost for external teams so wallets, exchanges, indexers, explorers, Agents, and application developers can participate.

Focus:

1. Developer quickstart.
2. Wallet and exchange integration.
3. L1/L2 Indexer APIs, L2 node-embedded indexing, and distributed L1 fact verification.
4. PWA Wallet adapter.
5. Multi-language STP clients.
6. Testnet tools and sample applications.

Current docs:

- [Developer Center](build/readme.md)
- [Developer Quickstart](build/quickstart.md)
- [Exchange and Wallet Integration](build/exchange-and-wallet.md)

## Phase 4: Ecosystem Growth

Goal: bring asset communities, developers, exchanges, institutions, and AI Agent teams into the SatoshiNet ecosystem.

Focus:

1. Builder Program.
2. GAS ecosystem narrative.
3. Asset-community and inscription-community integration.
4. Exchange and market maker partnerships.
5. Indexer / Explorer node ecosystem.
6. Official content, videos, tutorials, and community FAQ.
7. X and Telegram community building.

Current docs:

- [Ecosystem](ecosystem/readme.md)
- [Builder Program](ecosystem/builder-program.md)

## Long-Term Direction

SatoshiNet is designed to become the application layer for Bitcoin assets:

1. Asset facts are expressed by Bitcoin L1 and indexers.
2. Asset control is protected by STP.
3. Application execution is handled by SatoshiNet smart contracts.
4. Network resources are priced by GAS.
5. User experience is improved by wallets and AI Agents.
6. Ecosystem growth is driven by developers, asset communities, exchanges, indexers, and the broader community.

Every phase must produce verifiable evidence: transactions, code, testnet drills, APIs, documentation, and user-reproducible workflows.
