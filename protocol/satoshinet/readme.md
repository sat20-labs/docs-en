# SatoshiNet Protocol Overview

SatoshiNet is the Bitcoin-native extension network in the SAT20 ecosystem. Its goal is not to replace Bitcoin L1, but to provide a lower-cost, faster, contract-friendly, and Agent-friendly circulation environment for Bitcoin L1 assets while preserving user asset control.

## Positioning

SatoshiNet can be understood as a parallel execution network for Bitcoin assets:

1. Asset entry comes from Bitcoin L1.
2. Asset facts are recorded and verified by L1/L2 indexers.
3. Cross-layer asset control is provided by STP channels.
4. Daily transactions, contracts, and applications execute on SatoshiNet.
5. In abnormal situations, users can still protect assets through STP commitment transactions, force close, sweep, and punishment paths.

SatoshiNet is therefore not a custodial bridge and not an independent asset system unrelated to BTC. Its core promise is: assets come from Bitcoin, the safety boundary returns to Bitcoin, and the application experience happens on SatoshiNet.

## Relationship With Lightning

SatoshiNet inherits Lightning's key safety foundation: RSMC channels, commitment transactions, revocation, and punishment. The difference is that SatoshiNet is not centered on HTLC-routed payments. It combines RSMC channels with a parallel UTXO network and uses STP to support multi-asset, dynamic-capacity, and contract-enabled flows.

Key differences:

1. Channels can adjust capacity and asset sets through splicing-in / splicing-out.
2. Channels support BTC, ORDX, Runes, BRC20, and other Bitcoin L1 assets.
3. After assets enter SatoshiNet, they can circulate in L2 UTXOs and contracts.
4. Wallets and AI Agents can verify asset control through safety snapshots, commitment transactions, and punish coverage.

## Node Roles

| Node | Responsibility |
| --- | --- |
| Bootstrap Node | Helps Core Node discovery and admission |
| Core Node | Provides STP service, opens private channels with wallets, co-signs channel transactions, maintains channel state, and runs or configures an L1 indexer |
| Mining Node | Participates in SatoshiNet block production but does not provide STP service |
| Wallet Client | Ordinary user wallet or light client; connects to Core Nodes and holds private keys, channel state, and safety material |

Ordinary users do not need to stake assets when connecting to a Core Node and opening a private channel. Staking applies only when a node prepares to become a Core Node and connects to a Bootstrap Node for network service admission.

## Asset Model

SatoshiNet uses enhanced UTXO (enUTXO) to express assets. Unlike Bitcoin L1 UTXOs, which directly express only satoshi value, enUTXO can explicitly carry asset information such as ORDX, Runes, BRC20, or other assets recognized by indexers and entered through STP.

The model is designed to:

1. Let wallets and explorers directly see assets inside a UTXO.
2. Let contracts and transaction validation read asset information directly.
3. Let L1/L2 asset evidence be traced through UTXO, ascend, descend, and channel state.
4. Let AI Agents judge whether an asset is in a personal address, channel address, contract address, or pending state.

Plain sats on Bitcoin L1 still obey dust and fee constraints. On SatoshiNet, UTXOs can carry 0-sat assets, especially for BRC20 and Runes. ORDX must bind sats and needs `bindingSat` calculation.

## Indexers

Indexers are the foundation of SatoshiNet asset facts.

The L1 indexer parses Bitcoin UTXOs, sat ranges, Ordinals, Runes, BRC20, ORDX, confirmations, mempool, and reorg. Core Nodes need their own configured L1 indexer, so the Core Node network pushes distributed factual L1 indexer deployment.

The L2 indexer is integrated into SatoshiNet nodes and parses SatoshiNet UTXOs, ascend, descend, channels, contracts, transaction results, and asset state. Node operators maintain their own L2 state view, so L2 indexing is naturally distributed.

SatoshiNet safety needs STP + indexer together: STP provides control and exit paths; indexers provide asset facts and cross-layer evidence.

## Consensus and Execution

SatoshiNet evolves from a btcd-style UTXO system and extends it around SAT20 assets, channels, and contracts. It uses a consensus and block-production mechanism more suitable for an L2 execution environment, aiming to reduce transaction cost, improve confirmation speed, and let assets and contracts operate under Bitcoin ecosystem semantics.

The long-term direction is not to make Bitcoin L1 carry complex computation. Bitcoin L1 remains the final settlement and asset foundation; SatoshiNet carries higher-frequency and more complex application execution.

## Channel Contracts, Smart Contracts, and GAS

SatoshiNet contract capabilities have two categories: channel contracts and smart contracts.

Channel contracts are closer to the L1/L2 connection. They manage public asset pools and coordinate user-triggered cross-layer actions, such as public asset transcendence, launch, distribution, refund, and withdrawal. They are different from private STP channels: assets in contract pools do not belong to either channel peer and do not have the same user commitment/punishment safety mechanism.

Smart contracts run in the global SatoshiNet execution environment and rely on contract addresses, VM state, canonical Result TX, state roots, and GAS. They are for open application development such as AMMs, limit orders, prediction, EVM contracts, and natural language contracts.

GAS is the economic entry point for contract execution, transaction processing, and ecosystem incentives.

See [Channel Contracts](../channel-contracts/readme.md), [Smart Contract Protocol](../contracts/readme.md), and [GAS Ecosystem Opportunity](../../ecosystem/gas.md).

## Safety Model

SatoshiNet safety comes from multiple layers:

1. Bitcoin L1 provides asset origin, UTXO settlement, and the final channel exit boundary.
2. STP provides RSMC commitment transactions, revocation, CSV delay, force close, and punishment.
3. L1/L2 indexers provide verifiable evidence for asset facts, ascend/descend, channels, and contracts.
4. SatoshiNet nodes validate L2 transactions, UTXOs, and contract execution.
5. Wallets and AI Agents judge user control through safety snapshots, commitment export, punish coverage, and chain queries.

Users do not need to understand every detail, but wallets and Agents must understand them and provide verifiable safety conclusions before value movement.
