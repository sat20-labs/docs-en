# Third-Party STP Client Integration Guide

This guide is for wallets, SDKs, PWA adapters, CLIs, backend services, and AI Agent tools that want to implement an STP client. The goal is language-independent interoperability with a compatible Core Node.

This guide only covers STP client interoperability. Wallet creation, mnemonic import/export, password changes, and ordinary asset sends belong to the SAT20 Wallet or SAT20 Agent Wallet adapter layer. See [SAT20 Agent Wallet](../../ai/sat20-agent-wallet/).

## Integration Goals

An STP client must be able to:

1. Discover and verify a Core Node.
2. Manage the user's channel identity, signatures, and local channel state.
3. Construct, sign, send, and verify STP protocol messages.
4. Query the Bitcoin L1 indexer and SatoshiNet L2 indexer.
5. Drive open, splicing-in, unlock, lock, lock-with-expand, splicing-out, close, force close, and punish flows.
6. Recover pending operations after network errors, indexer delay, peer offline events, or local restarts.

Every value movement should be brought back to verifiable evidence: transactions, UTXOs, commitment height, commitments, ascend/descend records, and punishment coverage. A Core Node response, balance display, or single indexer response is not enough by itself.

## Recommended Architecture

| Module             | Responsibility                                                                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| Wallet boundary    | Stores keys, mnemonic, and user authorization. This can be a PWA, hardware wallet, mobile wallet, or local secure wallet |
| STP engine         | Implements STP messages, commitment state, channel actions, and recovery                                                 |
| Transaction engine | Builds and verifies Bitcoin L1, SatoshiNet, commitment, punish, and sweep transactions                                   |
| Asset engine       | Parses BTC, ORDX, Runes, BRC20, and amount precision                                                                     |
| State store        | Persists channel data, commitment height, commitments, revocation material, and pending operations                       |
| Chain query        | Queries L1/L2 UTXOs, assets, transaction visibility, confirmations, ascend/descend, and channel state                    |
| Safety monitor     | Produces safety snapshots, commitment exports, punish status, force-close plans, and sweep plans                         |

For AI Agents, the recommended path is SAT20 PWA Wallet Adapter: the PWA stores keys and database state, while the Agent sends authorized JSON operations and reads safety evidence.

## Core Node Discovery

Before connecting to a Core Node, the client must verify that:

1. Wallet, Core Node, Bitcoin L1 indexer, and SatoshiNet L2 indexer are on the same network.
2. The Core Node public key comes from trusted discovery or explicit user configuration.
3. Protocol version, CSV parameters, service fees, and feature list are acceptable.
4. The client can independently query L1/L2 state and does not rely only on the Core Node's response.

A normal user connecting to a Core Node does not stake assets. Staking is only relevant when a node connects to a Bootstrap Node and prepares to become a Core Node.

Example discovery response:

```json
{
  "protocol": "stp",
  "version": "1",
  "chain": "testnet",
  "core_node_pubkey": "02...",
  "endpoint": "https://core-node.example/stp",
  "features": [
    "open",
    "splicing-in",
    "unlock",
    "lock",
    "lock-with-expand",
    "splicing-out",
    "close",
    "force-close",
    "punish"
  ],
  "csv_delay": 1000
}
```

## Message Envelope

Authenticated STP messages should use a deterministic envelope:

```json
{
  "protocol": "stp",
  "version": "1",
  "chain": "testnet",
  "message_type": "unlock_request",
  "request_id": "uuid-or-monotonic-id",
  "timestamp": 1760000000,
  "channel_id": "tb1q...",
  "commit_height": 12,
  "sender_pubkey": "02...",
  "payload": {},
  "signature": "hex-signature"
}
```

Signing rules:

1. All fields except `signature` are signed.
2. Serialization must be deterministic, such as canonical JSON or an equivalent fixed encoding.
3. The receiver verifies sender key, channel identity, chain ID, and message type.
4. Any message that changes commitment state must include the current `commit_height`.
5. The client rejects height rollback, chain mismatch, invalid signatures, and asset amount inconsistency.

## Operation Interface

The client should expose a language-independent JSON operation interface to wallets, CLIs, and Agents. It should hide asset UTXOs, fee UTXOs, and channel-internal inputs. Coin selection and transaction package construction are wallet responsibilities.

### `stp.status`

Returns Core Node status, channel list, commitment height, pending operations, and indexer synchronization state.

```json
{
  "op": "stp.status",
  "chain": "testnet"
}
```

### `stp.open`

Opens a private channel with a Core Node.

```json
{
  "op": "stp.open",
  "chain": "testnet",
  "core_node": "https://core-node.example/stp",
  "amount_sats": 50000,
  "memo": "optional"
}
```

The client internally selects or constructs the L1 funding input. Normal user open does not require staking.

### `stp.splicing_in`

Brings Bitcoin L1 assets into the channel.

```json
{
  "op": "stp.splicing_in",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "asset": "runes:EXAMPLE",
  "amount": "1000"
}
```

The client selects asset inputs and ordinary BTC fee inputs internally. For BRC20, it can create a transfer inscription if no suitable transfer UTXO exists. The Agent passes asset and amount, not raw input lists.

### `stp.expand`

Adds assets already located at the channel address into channel management.

```json
{
  "op": "stp.expand",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "asset": "brc20:demo",
  "amount": "100"
}
```

Expand is used after interrupted splicing-in, rebuild, or direct transfer into the channel address. The client must use L1/L2 indexer evidence to decide whether ascend is needed and must not double-issue SatoshiNet assets.

### `stp.unlock`

Releases channel assets to the user's SatoshiNet address.

```json
{
  "op": "stp.unlock",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "asset": "ordx:demo",
  "amount": "100",
  "to": "tb1p..."
}
```

Unlock does not require user-provided fee rate or fee UTXO. SatoshiNet does not use Bitcoin L1 fee-rate semantics for this operation.

### `stp.lock`

Locks SatoshiNet personal-address assets back into the channel.

```json
{
  "op": "stp.lock",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "asset": "ordx:demo",
  "amount": "100"
}
```

The client chooses spendable L2 UTXOs internally and verifies that the asset has moved from pending into spendable state.

### `stp.lock_with_expand`

Restores channel control when capacity is insufficient.

```json
{
  "op": "stp.lock_with_expand",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "asset": "ordx:demo",
  "amount": "100"
}
```

This is a user-safety operation. It lets the client regain channel coverage without manually exiting to L1 and re-entering.

### `stp.splicing_out`

Exits channel assets to Bitcoin L1.

```json
{
  "op": "stp.splicing_out",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "asset": "brc20:demo",
  "amount": "100",
  "to_l1_address": "tb1p..."
}
```

The adapter handles asset inputs, transfer inscriptions, de-anchor, and fee inputs internally. If ordinary BTC fee funds are insufficient, it should return a clear error and next action.

### `stp.close`

Cooperatively closes or force-closes a channel.

```json
{
  "op": "stp.close",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "mode": "cooperative"
}
```

If cooperative close is impossible, the client must expose a force-close plan before broadcasting.

## Safety Interfaces

Every production-quality client should expose:

| Operation               | Purpose                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| `stp.safety_snapshot`   | Returns channel point, commitment height, commitment availability, balances, CSV, and punishment coverage |
| `stp.commitment_export` | Exports current commitment transactions and read-only verification data                                   |
| `stp.punish_status`     | Lists revoked remote commitments and punishment coverage                                                  |
| `stp.punish_build`      | Builds and dry-runs a punishment transaction for a given old commitment                                   |
| `stp.punish_broadcast`  | Broadcasts a verified punishment transaction                                                              |
| `stp.force_close_plan`  | Produces the local commitment, CSV delay, and sweep conditions                                            |
| `stp.sweep_build`       | Builds and verifies a sweep transaction after CSV                                                         |
| `stp.transaction`       | Tracks pending reservation, txids, channel state, and next action                                         |

Agents must treat `PUNISH_COVERAGE_UNKNOWN` and `PUNISH_COVERAGE_MISSING` as blockers for value movement.

## Successful Response Shape

Value-moving operations should return both user-facing status and verifiable evidence:

```json
{
  "ok": true,
  "op": "stp.unlock",
  "channel_id": "tb1q...",
  "commit_height_before": 12,
  "commit_height_after": 13,
  "txids": {
    "l1": [],
    "l2": ["..."],
    "commitment": ["..."]
  },
  "safety": {
    "status": "READY_SAFE",
    "punish_coverage": "COVERED"
  },
  "next": {
    "action": "poll",
    "op": "stp.transaction",
    "reservation_id": "..."
  }
}
```

If the result is unknown, return a structured pending state instead of a generic failure.

## Commitment Balance vs Spendable UTXO

Clients must separate commitment balances from spendable SatoshiNet UTXOs.

`local_balance` and `remote_balance` describe the current commitment allocation. `l2_spendable_balance` describes UTXOs the wallet can spend on SatoshiNet. A freshly spliced-in asset may already be part of the commitment, while its L2 anchor output is still pending. In that case, unlock or lock must wait.

## Asset Rules

| Asset      | Rule                                                                                                            |
| ---------- | --------------------------------------------------------------------------------------------------------------- |
| Plain sats | Bitcoin L1 has dust and fee constraints. SatoshiNet L2 does not have the same dust limit                        |
| ORDX       | ORDX binds to sats. Ordinals NFT-like `ordx:o` objects are ignored by STP ascend                                |
| Runes      | Runes can be carried by zero-sat L2 UTXOs. L1 still follows Bitcoin output constraints                          |
| BRC20      | BRC20 can require transfer inscription packages on L1. The adapter chooses or creates transfer UTXOs internally |

If a UTXO carries multiple assets, only the explicitly requested asset ascends. Other assets are excluded from the expected L2 balance.

## Unknown-Result Recovery

Timeout, EOF, connection reset, service unavailability, and temporary indexer lag are unknown results, not definite failures.

Recovery sequence:

1. Stop resubmitting the same request and lock related inputs.
2. Persist request JSON, reservation, txids, channel id, asset, amount, and error text.
3. Query pending reservation.
4. Query related Bitcoin L1 and SatoshiNet txids.
5. Query Core Node channel status, commitment height, channel point, and pending status.
6. If any transaction is visible, any side advanced state, or the reservation exists, keep tracking the original operation.
7. Retry only after both sides remain at the same old safe state, no pending reservation exists, and all related transactions remain invisible.

## Testnet Interoperability

Before connecting to mainnet, a third-party client should complete these testnet flows:

1. Open a private channel as a normal user without staking.
2. Read a safety snapshot after open.
3. Unlock and lock plain sats.
4. Splicing-in, unlock, and lock at least one Runes asset.
5. Splicing-in, unlock, and lock at least one BRC20 asset.
6. Splicing-out at least one protocol asset to Bitcoin L1.
7. Recover from unknown network results without double-spending inputs.
8. Prove punishment coverage after commitment height changes.
9. Generate a force-close plan when peer is unavailable.
10. Restart the client and recover pending reservations and safety state.

Mainnet release requires stricter long-running, reorg, indexer-delay, peer-offline, duplicate-request, database-recovery, and authorization-cancel tests.
