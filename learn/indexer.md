# Indexer: Bitcoin Asset Fact Layer

Bitcoin L1 natively understands only UTXOs and BTC. Ordinals, Runes, BRC20, ORDX, and other asset protocols are built on Bitcoin transactions, scripts, inscriptions, sat ranges, or protocol events. Users, wallets, exchanges, STP clients, and AI Agents need a unified asset fact layer before they can safely operate those assets.

The indexer turns Bitcoin L1 transaction facts into asset facts:

1. Which UTXOs an address owns.
2. Which BTC, Ordinals, Runes, BRC20, ORDX, or other assets each UTXO contains.
3. Whether those assets are confirmed and whether they may be affected by mempool, reorg, or protocol rules.
4. Whether a transfer, mint, inscription, deployment, or burn event is valid.
5. Whether an L1 transaction corresponds to an ascend, descend, or channel state on SatoshiNet.

## Why SatoshiNet Needs Indexers

STP brings Bitcoin L1 assets into channels and preserves user exit capability. The indexer tells wallets and Agents what those Bitcoin L1 assets are, where they are, whether they are confirmed, and whether they can enter a channel.

Without an indexer, STP cannot reliably answer:

1. Whether the user's selected UTXO actually carries the target asset.
2. Whether a BRC20 transfer UTXO is valid.
3. Whether Runes, ORDX, and Ordinals ownership matches what the user sees.
4. Whether a splicing-in L1 transaction is confirmed and can be mapped to SatoshiNet.
5. Whether assets have returned to a user-controlled L1 address after splicing-out or close.

The asset foundation of SatoshiNet is therefore STP + indexer. STP provides cross-layer control; the indexer provides asset facts and verifiable state.

## Indexers Do Not Control Assets

An indexer is not a custodian and is not a single trust point for asset safety. Its job is to compute asset state from public chain data and provide results to wallets, exchanges, browsers, STP clients, and Agents.

A qualified indexer should satisfy three requirements:

1. Recomputable: results come from Bitcoin L1 and SatoshiNet transactions, so third parties can independently run the same rules.
2. Traceable: balances and ownership can be traced to txid, vout, sat range, inscription, rune event, or protocol event.
3. Abnormal-state aware: mempool, confirmations, reorg, old states, invalid transfers, and protocol boundaries are explicit.

## L1 Indexer and L2 Indexer

SatoshiNet needs two indexing layers:

| Indexer | Role |
| --- | --- |
| Bitcoin L1 Indexer | Parses Bitcoin UTXOs, sat ranges, Ordinals, Runes, BRC20, ORDX, and ordinary BTC assets |
| SatoshiNet L2 Indexer | Parses SatoshiNet UTXOs, ascend, descend, channels, contracts, transaction execution, and asset state |

Wallets, exchanges, and Agents need to connect L1 and L2 evidence instead of reading one balance field: which Bitcoin UTXO an asset came from, which ascend event mapped it into SatoshiNet, where it is now, and which descend / L1 output represents it when it exits.

## Toward Distributed Indexers

SAT20 indexer is not a centralized API service. It is a set of independently runnable and cross-verifiable asset fact computation rules.

On SatoshiNet, the L2 indexer is already integrated into SatoshiNet nodes. Each node operator can maintain a local view of L2 transactions, UTXOs, ascend, descend, channels, and contracts. In this sense, L2 indexing is naturally distributed across the node network.

The L1 indexer reads Bitcoin L1. Core Nodes are expected to configure and rely on their own L1 indexers to verify L1 UTXOs, asset protocol events, confirmation state, and cross-layer transactions. Even when users access public APIs, the Core Node network itself creates a de facto distributed L1 indexer verification structure.

The long-term goal is not to create a new consensus layer, but to let more nodes independently compute and verify the same asset facts, reducing single-point errors, latency, forks, and data pollution.

## Relationship With AI Agents

When operating assets, an AI Agent cannot trust only a wallet balance display. It needs indexer evidence and should answer:

1. Which L1 UTXO the asset comes from.
2. Whether that UTXO is confirmed and unspent.
3. Whether the asset protocol event is valid.
4. Whether the asset has ascended to SatoshiNet or still remains on Bitcoin L1.
5. Whether channel commitments, L1/L2 indexers, and local wallet state are consistent.

This is why indexers are part of the SAT20 documentation core: they are the shared foundation for STP safety, exchange integration, explorer display, and AI Agent verification.
