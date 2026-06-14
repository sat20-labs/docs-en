# SAT20 Agent Wallet Interoperability Skill Specification

This specification describes how an AI Agent uses SAT20 Wallet and STP channels through a standardized adapter. It is not runtime code and not a local `SKILL.md`; it defines the protocol operations that the canonical SAT20 Agent Wallet skill uses.

The canonical skill lives in:

```text
https://github.com/sat20-labs/docs/tree/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet
```

## Skill Goals

An Agent using SAT20 Agent Wallet can:

1. Identify whether assets are on Bitcoin L1, a SatoshiNet personal address, or an STP channel.
2. Determine whether a usable channel exists and whether it is safe.
3. Choose open, splicing-in, unlock, lock, lock-with-expand, splicing-out, close, or recovery based on user intent.
4. Track operations through `txId` and `resvId`.
5. Stop safely during unconfirmed state, peer offline, locked UTXO, busy channel, insufficient fee, or missing safety evidence.

## Preferred Interface

Agents use high-level STP client operations rather than constructing low-level peer negotiation messages.

The high-level adapter automatically handles coin selection, transaction construction, signing, state persistence, broadcast, and peer messaging. Low-level messages are for client-library implementers and protocol debugging only.

## Preflight Checks

Before moving value, the Agent checks:

| Check | Purpose |
| --- | --- |
| Wallet exists and is unlocked | Signing requires wallet authorization |
| STP service is ready | Stop if L1/L2 sync or channel monitor is not ready |
| Network `chain` / `env` | Prevent mainnet/testnet mixing |
| L1/L2 indexers are reachable | Asset and confirmation state depend on indexers |
| Core Node is online | Channel cooperation needs a peer |
| Ready channel exists | Determines whether to open or operate |
| Asset name and divisibility | Amounts must match asset precision |
| UTXOs are not reservation-locked | Prevent duplicate spends |
| Operation involves BRC20 or Runes | These assets can require inscription or long confirmation |

## Safety Gate Before Value Movement

STP safety comes from commitments and punishment transactions, not from Core Node reputation. After open, every value movement requires a safety snapshot.

Minimum snapshot fields:

| Field | Agent check |
| --- | --- |
| `channelId` / `chanPoint` | Current commitment input points to the latest channel point |
| `commitHeight` | Monotonic; no rollback |
| `localCommitment` | User holds a broadcastable latest local commitment |
| `remoteCommitment` | Peer commitment can be identified and monitored |
| `localBalance` / `remoteBalance` | Balances match commitment outputs and asset roots |
| `csvDelay` | Force-close sweep window is acceptable |
| `punishCoverage` | Revoked peer commitments have punish transactions or construction material |
| `stateBackup` | Latest state and pending operations are persisted |

If these fields are missing, the Agent marks the channel `READY_DEGRADED`, stops splicing/unlock/lock/close, and reports `SAFETY_SNAPSHOT_REQUIRED` or `PUNISH_COVERAGE_MISSING`.

Punishment coverage states:

| State | Meaning | Agent action |
| --- | --- | --- |
| `NO_REVOKED_REMOTE_STATE` | No revoked peer commitment currently needs coverage | Continue, then re-check after every state advance |
| `COVERED` | Every revoked remote commitment has verified or broadcastable punish coverage | Continue normal value movement |
| `PUNISH_COVERAGE_UNKNOWN` | Adapter cannot prove coverage | Stop value movement |
| `PUNISH_COVERAGE_MISSING` | Revoked commitment exists but punish transaction is missing | Stop and enter recovery |

`READY_SAFE` also does not mean every asset UTXO is spendable. A freshly spliced-in or expanded asset can appear in commitment balance while its SatoshiNet anchor output is still pending. Unlock/lock waits for `UtxosL2` or `l2_spendable_balance`.

## Decision Trees

### Bring Bitcoin L1 Assets Into SatoshiNet

1. If no channel exists, open a plain-sat channel and wait for ready.
2. If assets are already at the channel address but not managed, run expand.
3. If assets are at the user's L1 address, run splicing-in.
4. Track transaction status and SatoshiNet asset summary.

### Recover a Closed Channel With Assets

1. Query L1 UTXOs, SatoshiNet assets, and channel ledger for the channel address.
2. If ledger evidence proves an existing client-Core channel, use reopen.
3. If minimum channel sats are missing, let the reopen path create new user funding.
4. If funding is visible but unconfirmed, poll funding, reservation, and local/Core channel status without retrying.
5. After both sides are ready, run `stp.safety_snapshot`.

### Use Channel Assets on SatoshiNet

1. Query channel readiness.
2. Confirm the asset is both in channel balance and spendable in L2 UTXOs.
3. Run unlock.
4. Wait for confirmation.
5. Query personal-address SatoshiNet balance.

### Put SatoshiNet Assets Back Under Channel Protection

1. Query channel readiness.
2. Query personal-address asset balance.
3. If capacity is sufficient, run lock.
4. If capacity is insufficient, run lock-with-expand.
5. Track reservation until completion.

### Exit SatoshiNet Assets to Bitcoin L1

1. If the asset is in the channel, run splicing-out to a valid Bitcoin address.
2. If the asset is on a SatoshiNet personal address, lock or lock-with-expand first, then exit through the channel.
3. For permanent exit, prefer cooperative close; use force close only when peer cooperation fails.
4. Track both SatoshiNet descend state and Bitcoin L1 confirmation.

## Asset Operation Rules

1. The Agent provides asset name, amount, and destination. It does not choose asset inputs or fee inputs.
2. The adapter internally chooses ordinary BTC fee inputs for protocol-asset splicing-out.
3. If fees are insufficient, the adapter returns a clear error and funding suggestion.
4. BRC20 transfer outputs are selected or created by the adapter.
5. If a BRC20 channel balance is non-zero after full splicing-out and asset UTXO evidence is invalid, the Agent treats it as an asset-consistency risk.
6. Multi-asset L1 UTXOs only ascend the explicitly requested asset. Other assets are ignored by the STP operation and excluded from L2 balance expectations.

## Request Field Guidance

Agent-facing adapter requests do not expose raw input UTXOs or fee UTXOs for splicing-in/out, lock, or unlock.

| Operation | Required user intent |
| --- | --- |
| `stp.open` | amount, optional memo |
| `stp.splicing_in` | channel, asset name, amount |
| `stp.splicing_out` | channel, asset name, amount, Bitcoin L1 destination |
| `stp.unlock` | channel, asset name, amount, optional SatoshiNet destination |
| `stp.lock` | channel, asset name, amount |
| `stp.lock_with_expand` | channel, asset name, amount |

Fee policy can be a high-level hint. The adapter may ignore it and use internal estimation.

## State Tracking

The Agent keeps polling reservations and chain state instead of trusting the initial return.

Rules:

1. If `resvId` is returned, track by `resvId`.
2. If only `txId` is returned, track chain confirmation and wallet action status.
3. `RS_CONFIRMED` means the protocol action reached confirmed state.
4. `RS_CLOSED` ends reservation lifecycle; it does not always mean the channel is closed.
5. `RS_FAILED` or `RS_REMOTE_FAILED` stops automatic progress and reports the error.
6. `CS_READY` is the usual precondition for channel operations.
7. Force-closing states block new ordinary channel actions.

## SatoshiNet UTXO Rules

1. SatoshiNet L2 UTXOs do not have Bitcoin dust limits and can have value `0`.
2. BRC20 and Runes do not need sat binding on L2.
3. On Bitcoin L1, BRC20/Runes transfer UTXOs still follow L1 output constraints.
4. ORDX always binds to sats, and `bindingSat` determines required carrier sats.
5. Plain-sat unlock amount is the released amount; the adapter handles SatoshiNet fee internally.
6. Plain-sat lock amount is the net amount locked into the channel; wallet inputs also cover SatoshiNet fee.

## Safety Guardrails

1. Mainnet value movement requires user confirmation.
2. No concurrent operations on a busy channel.
3. No retry after unknown result until reservation, txid, and channel state are checked.
4. No value movement with missing commitment or punishment coverage.
5. No mainnet use of testnet fault APIs.
6. No direct key or mnemonic exposure to the Agent.
