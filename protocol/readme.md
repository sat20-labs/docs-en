# SAT20 Protocol Stack

SAT20 is not a single protocol. It is a protocol, indexing, channel, and execution stack built around Bitcoin-native assets. Its goal is to let BTC, Ordinals, Runes, BRC20, ORDX, and other assets enter a network that is better suited for circulation, contracts, and AI Agent automation while preserving user control.

## Foundation Layers

| Layer | Role |
| --- | --- |
| Indexer | Parses Bitcoin L1 and SatoshiNet L2 transactions into queryable and auditable asset facts |
| STP | Uses RSMC channels, commitment transactions, revocation, and punishment to let assets enter, circulate, and exit SatoshiNet |
| SatoshiNet | Carries SatoshiNet transactions, enUTXO, channel contracts, smart contracts, and GAS economics |
| Channel Contracts | Manage public asset pools and coordinate user-triggered L1/L2 cross-layer actions |
| Asset Issuance Protocols | BTC, Ordinals, Runes, BRC20, ORDX, and other asset protocols unified by indexers |

The relationship can be summarized as:

1. BTC, Ordinals, Runes, BRC20, ORDX, and other protocols define or carry assets.
2. Indexers parse these assets from on-chain transactions into unified asset facts.
3. STP brings assets into a channel jointly controlled by the user and a Core Node, then creates the corresponding SatoshiNet asset state.
4. Channel contracts act as public facilities that coordinate user-triggered L1/L2 actions and public pool state.
5. SatoshiNet lets assets circulate quickly on L2, enter smart contracts, pay GAS, and return to Bitcoin L1 through STP when needed.

## Design Principles

1. Assets originate from Bitcoin L1. SatoshiNet does not create balances unrelated to Bitcoin L1.
2. Users retain exit capability. If a Core Node fails or behaves maliciously, users can rely on commitment transactions, punishment transactions, and force-close paths.
3. Asset facts are auditable. Wallets, exchanges, explorers, and AI Agents can trace L1/L2 evidence for each asset.
4. Protocols are language-independent. STP clients, indexer clients, and contract tools can be implemented in any language.
5. Agent-verifiable results. Protocol results include txid, UTXO, commit height, ascend/descend, punish coverage, and other evidence, not only balance changes.

## Reading Order

For protocol implementers:

1. [Indexer: Bitcoin Asset Fact Layer](../learn/indexer.md)
2. [STP Technical Whitepaper](stp/readme.md)
3. [SatoshiNet Protocol Overview](satoshinet/readme.md)
4. [Channel Contracts](channel-contracts/readme.md)
5. [Asset Issuance Protocols](indexer/asset-issuance.md)
6. [Smart Contract Protocol](contracts/readme.md)

Wallet, exchange, or AI Agent developers should start with the [Developer Center](../build/readme.md).
