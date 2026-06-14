# SAT20 Agent Wallet Asset Safety Control Guide

This guide is for AI Agents, wallet adapters, and third-party STP clients. It explains how an Agent can continuously answer: are the user's assets still inside a user-controlled safety boundary?

## Core Principle

STP safety does not come from trusting a Core Node. It comes from the user's chain-exit capability:

1. After assets enter a channel, Bitcoin L1 assets are locked in a 2-of-2 address controlled by the user and Core Node.
2. Every channel state change creates a new commitment state.
3. The user client holds the latest verifiable commitment transaction.
4. The user client releases old-state revocation material only after the new commitment is complete.
5. If the peer broadcasts a revoked commitment, the user client or watchtower can construct and broadcast a punishment transaction.
6. If the peer is offline or refuses cooperation, the user client can force close and sweep after CSV.

The Agent does not custody keys. It checks that these safety conditions hold and chooses the correct recovery action when they fail.

## Safety Material the Agent Must Understand

| Material | Purpose |
| --- | --- |
| channel id / channel address | Identifies the channel and 2-of-2 control address |
| channel point | Bitcoin L1 funding outpoint spent by commitments |
| commit height | Current commitment-state height; must be monotonic |
| local commitment tx | Latest user-side force-close transaction |
| remote commitment tx | Peer-side commitment monitored for old-state fraud |
| local / remote balances | Asset ownership in current state |
| CSV delay | Delay before local force-close output can be swept |
| revocation data | Punishment material for revoked states |
| punish tx set | Pre-signed or constructable punishment transactions |
| sweep capability | Ability to sweep after force close |
| Merkle roots | Integrity proof for channel asset state |

These materials must be persisted. The Agent can read summaries and transaction data but does not receive private keys, mnemonics, or unauthorized revocation secrets.

## Agent Safety State Machine

| State | Meaning | Agent action |
| --- | --- | --- |
| `NO_CHANNEL` | No channel exists | Open is allowed |
| `OPENING` | Funding broadcast but unconfirmed | Record txid and poll; no splicing/lock/unlock |
| `READY_SAFE` | Channel is usable and commitment/punishment coverage is complete | Ordinary operations allowed |
| `READY_DEGRADED` | Channel may be usable but safety material or backup is incomplete | Stop value movement and repair safety evidence |
| `PEER_OFFLINE` | Peer unavailable | Stop cooperative actions and prepare force close |
| `FORCE_CLOSING` | Local commitment broadcast | Poll CSV and prepare sweep |
| `REMOTE_COMMIT_SEEN` | Peer commitment seen on L1 | Determine whether it is latest or revoked |
| `PUNISH_REQUIRED` | Old commitment on-chain and punishment material exists | Broadcast punish |
| `UNSAFE` | Latest commitment or punishment ability cannot be proven | Stop and report risk |

`READY_SAFE` proves commitment and punishment safety. It does not automatically prove every L2 asset output is spendable. The Agent checks commitment balance separately from `UtxosL2` / `l2_spendable_balance`. Assets in `pendingUtxosL2` / `l2_pending_balance` cannot be used for unlock.

## Safety Checklist

Before every value movement, the Agent:

1. Calls `stp.safety_snapshot` or `stp.status`.
2. Confirms channel status is operable.
3. Confirms `commit_height` did not roll back.
4. Confirms local and remote commitment transactions exist.
5. Confirms local commitment input equals channel point.
6. Confirms commitment outputs match channel asset balances.
7. Confirms CSV delay is acceptable.
8. Confirms all revoked remote commitments have punishment coverage.
9. Confirms channel state backup is present.
10. For unlock/lock, confirms target L2 asset UTXOs are spendable, not pending.
11. Stops if any check fails.

If `punish_status` cannot provide a verifiable result, the correct state is `PUNISH_COVERAGE_UNKNOWN`, not "safe". Ordinary value movement stops until the wallet or STP adapter exports verifiable coverage.

If the adapter explicitly reports no revoked remote commitment, the Agent records `NO_REVOKED_REMOTE_STATE`. After the next state advance, it checks again. If every revoked commitment has `verified=true` and `broadcastable=true`, the Agent records `COVERED`.

After every value movement, the Agent confirms the commit height increased, the new local commitment is available, old remote commitment punishment coverage was saved or can be built, old-state revocation material was released only after the new state completed, and the new safety snapshot hash was recorded.

## Required Safety Operations

### `stp.safety_snapshot`

Reads a channel safety summary. It does not move assets and does not require signing.

Minimum response:

```json
{
  "op": "stp.safety_snapshot",
  "chain": "testnet",
  "channel_id": "tb1q...",
  "status": "READY_SAFE",
  "commit_height": 12,
  "chan_point": "txid:0",
  "csv_delay": 2,
  "local_commitment_txid": "...",
  "remote_commitment_txid": "...",
  "local_balance": [],
  "remote_balance": [],
  "l2_spendable_balance": [],
  "l2_pending_balance": [],
  "punish_coverage": {
    "state": "COVERED",
    "covered_revoked_commitments": 12,
    "missing": []
  },
  "state_backup": {
    "present": true,
    "last_updated": 1780000000000
  }
}
```

### `stp.commitment_export`

Exports read-only commitment verification data: local and remote commitment transaction, channel point, commit height, CSV delay, balances, commitment txid, and Merkle roots. It does not export keys or revocation secrets.

### `stp.punish_status`

Lists local punishment coverage for revoked remote commitments:

```json
{
  "op": "stp.punish_status",
  "channel_id": "tb1q...",
  "coverage": "COVERED",
  "revoked_commitments": [
    {
      "commit_txid": "...",
      "punish_txids": ["..."],
      "broadcastable": true,
      "verified": true
    }
  ]
}
```

### `stp.punish_build`

Constructs and verifies punishment transactions for a remote commitment seen on-chain. It returns signed punishment transactions or broadcast-ready summaries without exposing revocation secrets.

### `stp.punish_broadcast`

Broadcasts verified punishment transactions. This is a protective action, not an ordinary payment.

### `stp.force_close_plan`

Returns the local commitment, CSV delay, earliest sweep height/time, sweep construction conditions, and risk information when peer cooperation fails.

### `stp.sweep_build`

After CSV, builds, signs, and verifies a sweep transaction. The default mode is dry-run with `broadcast:false`; broadcast requires explicit wallet authorization.

## Testnet Punishment Drill

Testnet can expose fault-injection actions so an Agent can prove it can punish an old Core Node commitment. These actions are not mainnet protocol features.

The drill:

1. Retain the current Core Node-side commitment.
2. Advance the channel state so the retained commitment becomes revoked.
3. Ask the testnet Core Node to broadcast the retained old commitment.
4. Detect the old commitment on Bitcoin L1.
5. Build and verify punishment transactions.
6. Broadcast punishment transactions.
7. Mark the old channel as punished/closed and stop ordinary operations on it.

Testnet fault actions require runtime network checks, explicit unsafe-test confirmation, and mainnet rejection. PWA/mainnet wallet builds do not include server-side fault logic.
