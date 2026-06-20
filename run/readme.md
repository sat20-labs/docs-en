# Run the Network: Nodes and Infrastructure

An open network requires third parties to run nodes, indexers, explorers, RPC, and monitoring services. Operating infrastructure is not peripheral; it is the foundation of SatoshiNet's long-term decentralization, safety, and availability.

## Roles

| Role | Main Responsibility |
| --- | --- |
| Mining Node | Provides block production, transaction ordering, and contract execution |
| Core Node | Includes all Mining Node capabilities and additionally provides STP service, with higher online and channel-safety responsibility |
| L1 Indexer | Indexes multi-protocol Bitcoin L1 asset facts |
| L2 Indexer | Indexes SatoshiNet transactions, UTXOs, channels, contracts, and cross-layer states |
| Explorer / RPC | Provides verifiable access for users, wallets, exchanges, Agents, and developers |
| Monitoring and Operations | Provides backup, alerts, upgrades, and SLA capabilities |

## Current Focus

1. Clarify the role hierarchy: Core Node includes Mining Node capabilities and additionally provides STP service.
2. Clarify which GAS staking, fee, and penalty parameters remain under design.
3. Let third parties run nodes, indexers, or explorers on testnet.
4. Let wallets, DEXs, Agents, and exchanges verify state through independent infrastructure.
5. Publish which components are currently run by SAT20 Labs and which can be run by third parties.

**Page Status: In Development**
