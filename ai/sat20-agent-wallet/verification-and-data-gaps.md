# SAT20 Agent Wallet Verification Matrix and Data Gaps

This document is for AI Agents, PWA adapters, and third-party STP client implementers. It answers two questions:

1. How does an Agent independently verify that the user still controls asset safety?
2. What data, indexer records, or adapter interfaces are still needed when verification fails?

## Verification Principle

The Agent does not decide STP safety from a balance, a Core Node statement, or a single API response. Before every value movement, it combines four evidence groups:

1. Bitcoin L1 evidence: channel point, funding/splicing/commitment/punish transaction visibility, confirmation, and spent status.
2. SatoshiNet L2 evidence: anchor/deAnchor, unlock/lock transactions, L2 UTXO spendability, and asset amount consistency.
3. Local wallet evidence: latest local/remote commitment, commit height, CSV, punishment coverage, and persisted pending reservation.
4. Peer/Core Node evidence: peer channel status, commit height, and channel point compared with local state.

Only consistent evidence across these groups lets the Agent mark a channel `READY_SAFE`.

## Minimum Verification Matrix

| Scenario | What the Agent verifies | Required data | If missing |
| --- | --- | --- | --- |
| After open | Funding confirmed and commitments point to channel point | L1 funding tx, channel point, commitments, channel status | Mark `READY_DEGRADED`; poll only |
| After splicing-in | L1 tx confirmed and L2 anchor is spendable or explicitly pending | L1 tx status, L2 anchor tx, `pendingUtxosL2` / `UtxosL2` | Do not unlock that asset; keep polling |
| After expand | Channel-address assets are covered by latest commitment and not double-anchored | L1 UTXO, L2 ascend ledger, commit height, asset roots | Stop expand and require ledger evidence |
| Before unlock | Channel has asset, target asset is spendable on L2, punishment coverage is complete | Safety snapshot, L2 spendable UTXO, punish status | Do not unlock |
| Before lock | Personal-address L2 asset is spendable, channel capacity is enough or lock-with-expand is possible | Personal L2 UTXO, capacity, safety snapshot | Do not lock or plan lock-with-expand |
| Before splicing-out | Channel asset is sufficient, L1 destination is valid, adapter can construct safe fee and exit transactions | Safety snapshot, adapter preflight, deAnchor plan, L1 tx plan | Do not splicing-out |
| Peer offline | User holds latest local commitment and can plan force close and CSV sweep | Local commitment, CSV, sweep plan | Stop cooperative actions |
| Remote commitment on-chain | Determine whether it is latest or revoked | L1 commitment txid, current remote commitment, revoked index | Mark `UNSAFE` if unknown |
| Old commitment on-chain | Punish transaction is constructable, verifiable, and broadcastable | Revoked commitment, punish tx set, L1 visibility | Punish first; report missing coverage if incomplete |
| After punish | Punish tx is visible/confirmed and old channel is no longer used as normal channel | Punish txids, channel point spent status, fresh channel safety | Do not continue old channel |

## Agent Verification Steps

### 1. Build a Safety Snapshot

The Agent first calls `stp.safety_snapshot`. Minimum fields:

- `channel_id`
- `channel_address`
- `chan_point`
- `status`
- `commit_height`
- `csv_delay`
- `local_commitment_present`
- `remote_commitment_present`
- `local_balance`
- `remote_balance`
- `l2_spendable_balance`
- `l2_pending_balance`
- `merkle_roots`
- `punish_coverage`
- `state_backup`

If `stp.safety_snapshot` is unavailable, the Agent may temporarily infer from `stp.status` and channel detail, but production clients need a standardized snapshot.

### 2. Verify Commitments

The Agent verifies that local and remote commitments exist, commitment input points to the current channel point, balances match commitment output and asset roots, commit height did not roll back, and CSV delay matches user expectations.

Without commitment transactions, ordinary value movement stops because the user cannot prove unilateral exit ability.

### 3. Verify Punishment Coverage

The Agent normalizes `punish_coverage` into:

| State | Meaning | Action |
| --- | --- | --- |
| `NO_REVOKED_REMOTE_STATE` | No revoked remote commitment currently exists | Continue, then re-check after state advance |
| `COVERED` | Every revoked remote commitment has verified / broadcastable punish tx | Continue ordinary value movement |
| `PUNISH_COVERAGE_UNKNOWN` | Query failed or state is inconsistent | Stop ordinary value movement |
| `PUNISH_COVERAGE_MISSING` | Revoked commitment exists but punish tx is missing | Stop and recover |

Each revoked commitment should include `commit_txid`, `punish_txids`, `verified=true`, and `broadcastable=true`.

### 4. Verify L1/L2 Spendability

The Agent separates:

| Balance | Meaning |
| --- | --- |
| commitment balance | Asset ownership in latest commitment state |
| `l2_pending_balance` | Assets in anchor or pending L2 state |
| `l2_spendable_balance` | L2 UTXOs usable for unlock/lock |

Freshly spliced-in or expanded assets can appear in commitment balance while still pending on L2. The Agent waits for spendable L2 evidence before unlock/lock.

### 5. Verify Unknown Results

For timeout, connection interruption, or temporary service unavailable:

1. Query local reservation.
2. Query local/Core commit height.
3. Query related L1/L2 tx visibility.
4. If reservation exists, tx is visible, or either side advanced, keep polling and do not resubmit.
5. Retry only when no reservation exists, no tx is visible, and both sides remain in the same old `READY_SAFE` state.

## Data and Interface Gaps

| Gap | Why it matters | Proposed addition |
| --- | --- | --- |
| Standard `stp.safety_snapshot` | Agent needs one entry point for `READY_SAFE` | PWA adapter and third-party clients return the same JSON schema |
| Standard commitment export | Agent checks commitment input, output, txid, and asset ownership | `stp.commitment_export` returns local/remote commitment data, balances, roots |
| Standard punishment coverage | Agent needs per-revoked-commitment coverage | `stp.punish_status` returns `NO_REVOKED_REMOTE_STATE`, `COVERED`, `UNKNOWN`, or `MISSING` |
| L2 anchor spendable state | Agent must distinguish pending anchor from spendable UTXO | Snapshot includes `l2_spendable_balance` and `l2_pending_balance` |
| L2 channel ledger query | Rebuild/expand needs to know whether an L1 UTXO was anchored or returned by deAnchor | L2 indexer exposes channel ascend/deAnchor records, L1 txid/outpoint, and asset summary |
| deAnchor returned-output metadata | Splicing-out balance returned to channel address needs classification | descending v2 records L1 txid, returned channel output, asset, and amount |
| Force-close plan | Peer offline requires a clear exit path | `stp.force_close_plan` returns local commitment, CSV, and sweep condition |
| Sweep dry-run | Agent verifies sweep can be built without broadcasting | `stp.sweep_build` supports build/sign/verify and optional broadcast |
| Transaction polling | Network instability requires safe pending tracking | PWA adapter completes `wallet.transaction`, `stp.transaction`, and reservation queries |
| Explorer / indexer URL schema | Reports and videos need clickable evidence | Adapter returns L1/L2 explorer and indexer URLs for each txid |
| Safety backup status | Agent must know whether state and punishment material are persisted | Snapshot includes backup status fields |

## PWA Adapter Priority

The PWA wallet should prioritize:

1. `wallet.status`
2. `stp.status`
3. `stp.safety_snapshot`
4. `stp.transaction`
5. `stp.commitment_export`
6. `stp.punish_status`
7. `stp.punish_build` / `stp.punish_broadcast`
8. `stp.force_close_plan`

With these interfaces, an Agent can tell the user where assets are, whether the latest commitment exists, how to exit if the Core Node is offline, how to punish an old state, which assets are spendable, and which assets remain pending.

## User-Facing Verification Report

Example safe report:

```text
STP safety check: READY_SAFE

Channel: tb1q...
Channel point: <txid:vout>
Commit height: 8
Local commitment: present
Remote commitment: present
Punish coverage: COVERED
L2 spendable: brc20:f:sgas=100, runes:f:BITCOIN-TESTNET=10
L2 pending: empty

Conclusion: unlock is allowed. The wallet holds the latest commitment;
if the Core Node broadcasts an old state, the wallet has verified,
broadcastable punishment transactions.
```

Example blocked report:

```text
STP safety check: PUNISH_COVERAGE_UNKNOWN

Missing data: revoked remote commitment punishment coverage cannot be read.
The Agent will not execute splicing, unlock, lock, or close until the
wallet or STP adapter exports verifiable coverage.
```
