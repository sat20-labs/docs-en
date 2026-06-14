# STP Testnet Punish Drill Video Production Guide

This guide defines two video versions:

1. A one-minute public version for broad communication.
2. A complete technical version that can be longer and must explain old commitment broadcast, punish transaction construction, and punish broadcast.

The target audience understands Bitcoin, UTXOs, and channel safety. The video uses real testnet transactions and real wallet state. It does not use simulated txids as if they were real.

## Core Narrative

STP is not a custodial bridge. Users can bring BTC, Runes, BRC20, ORDX, and other Bitcoin-native assets into a channel, move them on SatoshiNet, and still hold latest commitments and punishment ability. If a Core Node broadcasts an old state, the user wallet can punish it.

Short version:

```text
We splice Runes and BRC20 assets from Bitcoin testnet4 into an STP channel,
unlock and lock them on SatoshiNet, then make the testnet Core Node broadcast
a revoked old commitment. The user wallet builds and broadcasts punish
transactions, proving that asset safety remains under user control.
```

## Required Sources

| Source | Purpose |
| --- | --- |
| SAT20 PWA Wallet | User wallet, channel, authorization, asset state |
| SAT20 Agent Wallet skill output | Standard JSON operations and verifiable responses |
| Bitcoin testnet4 explorer | Funding, old commitment, punish transactions |
| SatoshiNet explorer | L2 asset and channel transactions |
| L1 indexer API | Bitcoin testnet4 UTXO and asset information |
| L2 indexer API | SatoshiNet UTXO, asset, and channel-address state |
| `stp.safety_snapshot` | Commitment height, channel point, commitments, punishment coverage |
| `stp.splicing_in` / `stp.splicing_out` | How protocol assets enter and exit the channel |
| `stp.unlock` / `stp.lock` | How assets move between personal L2 address and channel protection |
| `stp.punish_build` / `stp.punish_broadcast` | Punishment construction and broadcast |

## One-Minute Structure

| Time | Visual | Message |
| --- | --- | --- |
| 0:00-0:06 | PWA wallet asset + channel page | User wallet connects to a Core Node private channel |
| 0:06-0:12 | Bitcoin testnet4 funding tx | 2-of-2 channel address is the L1 safety boundary |
| 0:12-0:24 | Indexer evidence + splicing tx | Runes, BRC20, and ORDX can enter or exit the channel |
| 0:24-0:36 | SatoshiNet explorer + unlock/lock tx | Assets move on L2 and can return to channel protection |
| 0:36-0:48 | `stp.safety_snapshot` | Agent verifies channel point, commit height, commitments, and punish coverage |
| 0:48-0:56 | Old commitment + punish tx | Old Core Node state is punished on Bitcoin testnet4 |
| 0:56-1:00 | Final PWA state | User keeps exit and punishment capability |

## Complete Technical Version

Suggested length: 3-5 minutes.

| Segment | Visual | Message |
| --- | --- | --- |
| 0:00-0:20 | PWA wallet | Assets are controlled by the wallet, not custodied by Core Node |
| 0:20-0:45 | Funding tx | BTC enters 2-of-2 channel address |
| 0:45-1:20 | Splicing and indexers | Runes, BRC20, and ORDX are handled through real L1/L2 evidence |
| 1:20-1:50 | Unlock / lock | Assets leave channel protection for L2 use and return to channel protection |
| 1:50-2:20 | Safety snapshot | Agent reads channel point, commit height, commitments, and punish coverage |
| 2:20-2:45 | Testnet retain old commitment | Retain Core Node-side commitment and then advance state |
| 2:45-3:15 | Old commitment broadcast | Show old commitment on Bitcoin testnet4 |
| 3:15-3:45 | Punish build dry-run | Agent builds punishment package without exposing secrets |
| 3:45-4:20 | Punish broadcast | Punish package is visible or confirmed on Bitcoin testnet4 |
| 4:20-5:00 | Final state | Old channel is closed/punished; fresh channel proves safety close-out |

## Transaction Explanation Template

Use one sentence per transaction:

| Transaction | Explanation |
| --- | --- |
| Funding tx | Locks Bitcoin L1 assets into a 2-of-2 channel address controlled by the user and Core Node |
| Splicing-in tx | Adds Bitcoin-native protocol assets to an existing channel |
| ORDX anchor tx | Maps ORDX assets into SatoshiNet according to `bindingSat`; small ORDX can use stub sats to satisfy L1 constraints |
| Pending L2 anchor output | Asset is in the commitment asset set, but its L2 output is not yet spendable |
| Unlock tx | Releases channel assets to the user's SatoshiNet address |
| Lock tx | Returns SatoshiNet personal assets to channel protection |
| Splicing-out tx | Exits channel assets to Bitcoin L1 |
| Commitment tx | Defines asset ownership at a specific channel height |
| Old commitment tx | Testnet-broadcast revoked Core Node-side commitment |
| Punish tx | User wallet spends old commitment outputs with saved punishment material |
| Sweep tx | After CSV, sweeps force-close outputs back to the user |

## Production Rules

1. Show real txids and shorten only visually.
2. Hide mnemonic, private key, password, and revocation secret.
3. Keep each key screen visible for 4-8 seconds.
4. Use technical captions without return or price claims.
5. Mark every video as Bitcoin testnet4 / SatoshiNet testnet.
6. Do not show network errors, retries, backend logs, or debugging. Show the Agent's final safe interpretation.

## Capture Flow

1. Wait for channel `READY_SAFE`.
2. Save `stp.safety_snapshot`.
3. Run or reference Runes `stp.splicing_in`.
4. Run or reference BRC20 `stp.splicing_in`.
5. Optionally show ORDX small-amount splicing-in.
6. Unlock one spendable asset.
7. Lock it back, or use lock-with-expand if capacity requires it.
8. Optionally run a small splicing-out.
9. Save another safety snapshot.
10. Retain server commitment on testnet.
11. Advance channel state.
12. Broadcast retained old commitment on testnet.
13. Build punish dry-run.
14. Broadcast punish package.
15. Show old channel closed/punished and fresh channel safety.
