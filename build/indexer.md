# Indexer Integration and Asset Fact Layer

This page is for wallets, exchanges, explorers, STP clients, AI Agents, and infrastructure teams. It explains the role of SAT20 indexers.

## What Is an Indexer?

The indexer is the asset fact layer for Bitcoin L1 and SatoshiNet L2. It does not custody assets, sign for users, or decide ownership. It computes and exposes asset state from on-chain transactions and protocol rules.

SAT20 needs at least two indexer types:

| Type | Main data |
| --- | --- |
| Bitcoin L1 Indexer | BTC UTXOs, sat ranges, Ordinals, Runes, BRC20, ORDX, inscriptions, mempool, confirmations, reorg state |
| SatoshiNet L2 Indexer | SatoshiNet UTXOs, ascend, descend, channels, contracts, transaction execution, asset state |

## Who Needs Integration

| Role | Purpose |
| --- | --- |
| Wallet | Display assets, select UTXOs, verify cross-layer operation results |
| STP client | Judge asset inputs and chain state for splicing-in/out, open/close, lock/unlock |
| Exchange | Deposits, withdrawals, reconciliation, risk control, and recovery |
| Explorer | Display L1/L2 transactions, assets, channels, and contract state |
| AI Agent | Independently verify asset evidence and safety boundaries before value movement |

## Minimum Capability

A qualified client should query:

1. Address UTXO list.
2. Asset details of a single UTXO.
3. Transaction confirmations, mempool state, and spent state.
4. Ordinals, Runes, BRC20, ORDX, and other protocol asset state.
5. L1 ascend / descend related transactions.
6. L2 address balances, UTXOs, channels, and contract transactions.
7. Explicit error state for reorg or not-yet-indexed data.

For STP, the key question is not "what is the balance?" but "is the asset evidence chain complete?" L1 UTXO, L1 transaction confirmation, ascend / descend event, L2 UTXO, channel commitment state, and wallet local state must explain each other.

## Integration Principles

1. A single balance field is not an asset safety proof.
2. For Bitcoin L1 assets, track txid, vout, asset protocol, confirmations, and spent state.
3. For BRC20 transfer, Runes, ORDX, Ordinals, and other assets, evaluate validity by protocol rules.
4. For STP operations, distinguish L1 confirmed, L2 ascended, L2 spendable, and channel updated.
5. For mempool, network error, EOF, not indexed, and reorg, return explicit states instead of defaulting to success or failure.
6. Exchanges and Agents should keep verifiable evidence: txid, vout, asset, amount, height, confirmations, indexer source, and query time.

## Relationship With STP

STP protects asset control; indexers provide asset facts.

When a user starts splicing-in, the indexer helps the client confirm whether the target UTXO carries the asset, whether the asset is valid, and whether the transaction is confirmed. STP brings that asset into the channel and creates the corresponding SatoshiNet state.

When a user starts splicing-out or close, STP constructs the exit path. The indexer helps wallets and Agents confirm whether L2 descend corresponds to the Bitcoin L1 output and whether the asset has returned to a user-controlled address.

## Distributed Indexer Direction

Current integrations can start with official APIs, but indexers should not become a single service. They should evolve into a multi-party, independently runnable, cross-verifiable asset fact network.

The L2 indexer is already integrated into SatoshiNet nodes. Node operators maintain their own L2 UTXO, ascend, descend, channel, contract, and transaction state, so L2 indexing naturally distributes with the node network.

The L1 indexer serves Bitcoin L1 asset facts. SatoshiNet Core Nodes configure and depend on their own L1 indexers to verify L1 UTXOs, asset protocol events, confirmations, and cross-layer operations. In this sense, the Core Node network pushes factual distributed deployment of L1 indexers.

Distributed indexer work includes:

1. Publicly reproducible rules.
2. Cross-verifiable data results.
3. Standardized reorg, mempool, and abnormal states.
4. Multiple data sources for exchanges, wallets, and Agents.
5. Critical asset states explained by evidence instead of a single API response.
6. Core Nodes, wallets, exchanges, and browsers independently running or selecting trusted L1/L2 indexers.

This gives multi-protocol Bitcoin L1 assets a safer and more unified expression, and helps SatoshiNet become a more trustworthy Bitcoin-native extension network.
