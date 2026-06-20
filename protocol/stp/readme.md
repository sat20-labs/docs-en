# STP Technical Whitepaper

STP, the Satoshi Transcending Protocol, is the asset-channel protocol that connects Bitcoin L1 with SatoshiNet. Its goal is not to create a custodial bridge. STP lets a user bring Bitcoin-native assets into a channel jointly controlled by the user and a Core Node, then use those assets on SatoshiNet with faster settlement, lower operating cost, and contract compatibility.

This document describes protocol semantics only. It does not depend on a specific codebase, class name, API framework, or programming language. Any wallet, SDK, PWA, CLI, or AI Agent adapter can implement an STP client from these rules and connect to a compatible Core Node.

## Summary

STP uses the same security foundation as the Lightning Network: RSMC, a 2-of-2 channel, versioned commitment transactions, revocation material, CSV delay, and punishment transactions. A user does not have to trust that the Core Node is always online or always honest. The user needs to hold the latest commitment transaction and must be able to punish the peer if an old state is broadcast.

STP extends that model in four directions:

1. Multi-asset support: BTC, ORDX, Runes, BRC20, and other Bitcoin-native assets can enter the channel.
2. Dynamic capacity: splicing-in, splicing-out, expand, and lock-with-expand can change the channel asset set.
3. SatoshiNet settlement: assets can move into SatoshiNet UTXOs, contracts, and the GAS economy.
4. Indexer-backed asset facts: the L1 indexer describes Bitcoin-side asset facts, while the L2 indexer describes ascend, descend, channel, and contract state.

The asset foundation of SAT20 should be understood as STP plus Indexer. The Indexer tells clients what an asset is, where it is, and whether it is valid. STP defines how those assets enter, move, exit, and stay under user control during failures.

## Protocol Goals

STP is designed to provide:

1. Final user control: channel assets are controlled by the user and Core Node together, while the user always holds a broadcastable latest commitment.
2. BTC L1 exit safety: if the Core Node is offline, refuses service, or acts maliciously, the user can force close and sweep.
3. Old-state punishment: if the Core Node broadcasts a revoked commitment, the user can construct a punishment transaction.
4. Multi-asset consistency: sats, ORDX, Runes, BRC20, and other assets keep consistent amounts and protocol rules across L1, the channel, and L2.
5. Verifiable asset facts: the client must verify UTXOs, assets, ascend, descend, confirmations, and channel state through L1/L2 indexers.
6. Language-independent interoperability: a client only needs to follow the message, signature, transaction, state, and recovery rules.

## SatoshiNet Network Roles

Older documents or APIs may use the term `STP Server`. At the protocol level, this is the STP service capability exposed by a Core Node.

| Role | Responsibility |
| --- | --- |
| Bootstrap Node | Assists Core Node discovery and admission |
| Core Node | Includes all Mining Node capabilities and provides STP service, opens private channels with wallets, co-signs channel transactions, maintains the service-side channel state, and runs or configures an L1 indexer |
| Mining Node | Produces SatoshiNet blocks, but does not provide STP channel service |
| Wallet Client | User wallet or client. It connects to a Core Node and holds keys, channel state, commitments, and punishment material |

A normal user connects to a Core Node and opens a private channel. This path does not require staking. Staking belongs to the path where a node connects to a Bootstrap Node and prepares to participate as a Core Node.

## Channel Model

An STP channel is a 2-of-2 asset-control relationship between a user and a Core Node. Bitcoin L1 UTXOs on the channel address define the on-chain control boundary. SatoshiNet L2 state represents how channel assets move inside SatoshiNet.

Every channel contains at least:

| Object | Meaning |
| --- | --- |
| Channel address | A 2-of-2 address derived from the user key and Core Node key |
| Channel point | The current Bitcoin L1 outpoint spent by the commitment transaction |
| Commitment height | The monotonic version number of channel state updates |
| Local commitment | The latest exit transaction the local side can broadcast |
| Remote commitment | The peer-side commitment version monitored for old-state fraud |
| Revocation material | Data needed to punish a revoked remote commitment |
| CSV delay | The force-close window during which the peer can punish |
| Pending operation | An unfinished open, splicing, lock, unlock, close, or recovery action |

The client must persist the latest commitment transaction, commitment height, revocation material, channel point, and pending operation after every commitment update. A displayed balance alone is not proof that the user still controls the assets.

## Asset Model

STP does not redefine Bitcoin L1 asset protocols. It places them inside a channel and SatoshiNet state model.

| Asset | STP rule |
| --- | --- |
| Plain sats | Bitcoin L1 has dust and fee constraints. SatoshiNet L2 can have zero-sat UTXOs |
| ORDX | ORDX binds to sats. `bindingSat` determines the required carrier sats. `ordx:o` objects do not ascend |
| Runes | L2 does not require sat binding and can carry Runes in zero-sat enUTXOs. L1 outputs still follow Bitcoin rules |
| BRC20 | L2 does not require sat binding. L1 enter/exit may require transfer inscriptions and related transaction packages |

If one L1 UTXO carries multiple ascendable assets, STP only ascends the asset explicitly requested by the operation. Other assets do not enter SatoshiNet. Internal coin selection should usually avoid multi-asset UTXOs; when it must use one, the user preview should make clear which assets will not ascend.

## Channel States

The protocol can be summarized with these abstract states:

| State | Meaning |
| --- | --- |
| `INIT` | Channel negotiation started |
| `FUNDING_BROADCASTED` | Bitcoin L1 funding transaction was broadcast |
| `FUNDING_CONFIRMED` | Funding transaction was confirmed |
| `ANCHOR_BROADCASTED` | SatoshiNet ascend / anchor transaction was broadcast |
| `ANCHOR_CONFIRMED` | Ascend / anchor was confirmed |
| `READY` | Normal value movement is allowed |
| `CLOSING` | Cooperative close in progress |
| `FORCE_CLOSING` | A commitment transaction was broadcast |
| `SWEEPING` | CSV expired and sweep is in progress |
| `CLOSED` | Channel close is complete |
| `PUNISHED` | A revoked state was punished and the channel is terminated |

Normal value movement should only start from `READY`. If there is a pending operation, unknown chain result, missing commitment coverage, or unsynchronized indexer state, the client should stop splicing, lock, unlock, and close actions until recovery converges.

## Protocol Operations

### Open

Open creates a private channel between the user and a Core Node. A normal user does not stake assets. The flow includes parameter negotiation, L1 funding, initial commitment signing, funding confirmation, L2 ascend / anchor, and channel readiness.

The client must verify that the channel address is derived from the user key and Core Node key, that funding pays to the correct channel address, that fees and change match user authorization, that the initial commitment is broadcastable by the user, that the Core Node signature is valid, and that the L1 funding and L2 ascend / anchor can be verified through indexers.

### Splicing-In / Expand

Splicing-in brings new Bitcoin L1 assets into an existing channel. It can include an L1 transfer into the channel address, L2 ascend / anchor, commitment update, and channel capacity update.

Expand is used when assets are already at the channel address but are not covered by the current commitment state. This can happen after an interrupted splicing-in, after channel recovery, or when the user has transferred assets directly to the channel address.

The user-facing adapter should not require the user or Agent to choose asset UTXOs or fee UTXOs. Coin selection and transaction package construction are wallet responsibilities. For BRC20, if no suitable transfer UTXO exists, the adapter may internally create the transfer inscription before splicing-in.

### Unlock

Unlock releases user-owned channel assets to the user's SatoshiNet address. After unlock, the assets are controlled by the user on L2 and can be transferred, used in contracts, or later locked back into the channel.

Unlock advances the commitment height. Before unlock, the client must verify that the channel is ready, no pending operation exists, the latest commitment exists, and punishment coverage is available. After unlock, it must verify that the commit height increased monotonically.

### Lock / Lock-With-Expand

Lock moves assets from the user's SatoshiNet address back into channel protection. After lock, the assets return to the RSMC commitment safety boundary.

Lock-with-expand is used when channel capacity or asset coverage is insufficient. It is an important user-control capability: it lets users bring assets back under channel protection without manually exiting and re-entering L1.

Lock and unlock should not require user-supplied input UTXOs or fee UTXOs. The adapter selects the L2 asset inputs internally and returns verifiable transaction evidence.

### Splicing-Out

Splicing-out exits channel assets back to Bitcoin L1. It normally includes a commitment update, L2 descend / de-anchor, L1 outputs, and indexer confirmation.

The client must verify that the asset, amount, and destination address match user authorization, that L2 descend and L1 outputs can be linked by indexer evidence, that the L1 output follows the asset protocol, and that fee inputs are legal. Except for the internal plain-sat unlock exception, initiator fees should not be paid from channel-address UTXOs.

### Close / Force Close / Sweep

Close is a cooperative channel close. Force close is a unilateral close by broadcasting the latest commitment transaction. Sweep moves funds after the CSV delay.

If the Core Node is unavailable, the client must produce a force-close plan: the local commitment that can be broadcast, the CSV delay, the sweep conditions, and the remaining risk. An Agent should not advise the user to keep relying on a channel if it cannot prove a commitment and sweep path.

### Punish

If the Core Node broadcasts an old remote commitment, the client identifies that commitment as revoked and constructs a punishment transaction with the corresponding revocation material.

After every commitment height increase, the client should update punishment coverage and report whether revoked remote commitments exist, whether each has a verified or broadcastable punishment transaction, whether the CSV window is still open, and what terminal channel state follows a punish broadcast.

Mainnet clients must not depend on testnet fault-injection interfaces. Testnet may expose old-commitment retention and broadcast actions so an Agent can prove that it can detect old states and broadcast punish transactions.

## Unknown Network Results

An STP client must distinguish explicit failure from unknown result. Timeout, EOF, connection reset, service restart, or temporary indexer lag must not be treated as proof that a transaction did not happen.

Recommended rules:

1. Before entering broadcast or final commitment exchange, persist the pending operation, channel snapshot, and related transaction IDs.
2. If a broadcast response is unknown, first query whether the L1/L2 transaction is visible.
3. If the transaction is visible, keep tracking confirmation and do not spend the same inputs again.
4. If the transaction is not yet visible, keep the inputs locked and the operation pending until monitoring converges.
5. Only retry after both sides are still in the same old safe state, no pending operation exists, and all related transactions remain invisible.

This applies to open, splicing-in, splicing-out, unlock, lock, close, and force close.

## Channel Recovery

An STP client should support:

| Capability | Scenario |
| --- | --- |
| Restore | Local process or database failed, but local backup or peer state can recover the latest channel |
| Reopen | Channel is closed, but the channel address still holds user assets and the same client-Core identity should be restored |
| Rebuild | Channel state was lost or the L1 channel point changed; L1/L2 ledger evidence is used to reassign assets |
| Expand | Assets at the channel address are not covered by the current commitment state |
| Clean stale channel | On-chain state is closed or punished, but a stale active channel remains locally |

Recovery must not double-issue SatoshiNet assets. Whether an L1 UTXO needs ascend should be decided from the L2 channel ledger ascend / descend records and the current L1/L2 indexer view. Assets that cannot be proven should not be automatically anchored.

## Minimum Client Requirements

A safe STP client needs:

1. Key management and message signing.
2. Core Node discovery and identity verification.
3. Bitcoin L1 and SatoshiNet query capability.
4. L1/L2 transaction construction, signing, verification, and broadcast.
5. Persistent channel state, commitments, revocation material, and pending operations.
6. Asset protocol parsing and amount precision validation.
7. Unknown-result recovery.
8. Safety snapshot, commitment export, punish status, force-close plan, and sweep build interfaces.

Interoperability documents:

| Document | Purpose |
| --- | --- |
| [STP Messages and Data Model](messages-and-data-model.md) | Defines the message families, shared fields, asset objects, commitment objects, and error semantics a third-party client needs |
| [STP Message Sequences](message-sequences.md) | Describes message order and validation points for open, splicing, lock, unlock, close, punish, and recovery flows |
| [Third-Party STP Client Integration Guide](client-integration.md) | Integration guidance for wallets, SDKs, PWA adapters, CLIs, and AI Agents |
| [Third-Party STP Client Implementation Checklist](implementation-checklist.md) | Capability checklist before a third-party client enters testnet or mainnet |

## Verifiable Safety for AI Agents

STP is designed so an AI Agent can judge whether assets are still under user control instead of merely calling APIs. Before moving value, an Agent should verify that:

1. The channel is ready.
2. The wallet holds the latest local commitment.
3. Revoked remote commitments have punishment coverage.
4. Commitment height increases monotonically.
5. L1/L2 indexer evidence matches wallet state.
6. No pending or unknown-result value movement exists.
7. User keys, mnemonic, and signing remain inside the wallet security boundary.

If these conditions fail, the Agent should stop normal operations and move to recovery, force close, punish, or human confirmation.
