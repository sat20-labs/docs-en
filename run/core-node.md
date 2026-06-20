# Core Node

A Core Node includes all Mining Node capabilities and additionally provides STP service, connecting user wallets with the SatoshiNet execution layer. It carries higher uptime requirements, channel-state responsibility, and cross-layer asset service responsibility.

## Relationship with Mining Nodes

| Dimension | Mining Node | Core Node |
| --- | --- | --- |
| Base capability | Block production, ordering, execution | Includes all Mining Node capabilities |
| Extra responsibility | Does not provide STP channel service | STP service, channel state, cross-layer asset control |
| Asset-safety responsibility | Related to network execution and consensus | Also directly related to user channels, commitments, and abnormal recovery |
| L1 Indexer dependency | Needs to understand chain facts | Needs more stable L1 Indexer access for STP cross-layer verification |
| Staking requirement | Planned GAS staking | Planned higher requirement; parameters remain under design |

## To Be Completed

1. Boundary between STP service fees and network fees.
2. Recovery boundary when user channels fail.
3. Public testnet admission flow.
4. Handling node offline, refusal of service, or state abnormalities.

**Page Status: Design in Progress**
