# Agent-Controlled Wallet Roadmap

This roadmap defines SAT20 PWA Wallet and SAT20 Agent Wallet capabilities from the shared perspective of users and AI Agents. The goal is to let an Agent safely control Bitcoin testnet/mainnet and SatoshiNet asset movement under user authorization, while explaining why STP and SatoshiNet are Bitcoin-native and safely exitable.

## Core Principles

User asset safety is the destination of every feature.

Users may not understand UTXO, commitment transactions, CSV, splicing, BRC20 commit/reveal, or Runes. The Agent must understand those constraints and turn them into safe operations:

1. Identify where assets are: Bitcoin L1, STP channel, SatoshiNet personal address, or unconfirmed transaction.
2. Explain risk before requesting authorization.
3. Simulate and verify before signing and broadcasting.
4. Preserve an exit, recovery, and proof path at all times.
5. When lock capacity is insufficient, use lock-with-expand to restore channel control.

## PWA Wallet as Human UI and Agent Executor

SAT20 PWA Wallet is both a user wallet and the secure execution layer for Agents.

| User       | PWA capability                                                                                |
| ---------- | --------------------------------------------------------------------------------------------- |
| Human user | Create/import wallet, view assets, confirm transactions, back up mnemonic, change password    |
| AI Agent   | Standard JSON adapter, permission session, operation preview, risk check, transaction polling |
| Developer  | Testnet adapter, reproducible scripts, error codes, operation playbooks                       |

Private keys, mnemonic, password input, signing, and revocation material stay inside the PWA. The Agent sends intent, reads state, and explains results.

## Skill System

SAT20 Agent Wallet contains four logical skill areas:

1. Wallet management: `wallet.status`, `wallet.create`, `wallet.import`, `wallet.export_mnemonic`, `wallet.change_password`, `wallet.send_assets`, `wallet.transaction`.
2. STP channel lifecycle: `stp.status`, `stp.open`, `stp.close`, `stp.splicing_in`, `stp.splicing_out`, `stp.unlock`, `stp.lock`, `stp.lock_with_expand`, `stp.transaction`.
3. Asset safety: network checks, channel readiness, commitment coverage, punishment coverage, force-close planning, and user-readable risk reports.
4. Testnet lab: wallet creation, test assets, open, splicing, unlock, lock, splicing-out, and punish drill.

The installable skill is maintained in the canonical docs repository:

```
https://github.com/sat20-labs/docs/tree/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet
```

## PWA Adapter Requirements

The PWA exposes an Agent-friendly adapter:

1. HTTP, postMessage, or local CLI wrapper transport.
2. Unified JSON envelope for all requests.
3. Operation preview for each value-moving action: asset, amount, source, destination, fee, risk.
4. `AUTH_REQUIRED` response so the Agent can guide the user to confirm in PWA.
5. Scoped authorization sessions: testnet-only, max amount, selected asset, selected operation.
6. Revocable Agent authorization.
7. Transaction and channel operation audit log.
8. Pending transaction polling.
9. Emergency exit and force-close guidance.
10. Automatic suggestion of lock-with-expand when capacity is insufficient.

## Implementation Priorities

| Priority | Task                                                              | Acceptance standard                                                                                 |
| -------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| P0       | Complete PWA `wallet.status`                                      | Real address, network, unlock state, L1/L2 assets, UTXO summary, WASM ready state                   |
| P0       | Complete PWA `stp.status`                                         | Core Node, channel list, status, commit height, pending reservation                                 |
| P0       | Complete `wallet.transaction` / `stp.transaction`                 | Agent can poll txid/reservation without reading backend logs                                        |
| P0       | Standardize PWA authorization modal                               | Every value movement shows asset, amount, source, destination, fee, channel, and risk               |
| P0       | Productize `stp.safety_snapshot`                                  | Agent reads channel point, commit height, commitments, balances, CSV, punishment coverage           |
| P0       | Unknown-result handling                                           | Timeout or service unavailable returns `NETWORK_RESULT_UNKNOWN`, reservation, txid, and next action |
| P1       | `stp.commitment_export`                                           | Export read-only commitments and verification data without keys or revocation secrets               |
| P1       | `stp.force_close_plan`                                            | Peer offline returns local commitment, CSV wait, and sweep condition                                |
| P1       | `stp.punish_status` / `stp.punish_build` / `stp.punish_broadcast` | Agent can prove, build, and broadcast punishment transactions                                       |
| P1       | Split `stp.sweep_build`                                           | Build / sign / verify / optional broadcast with stable schema                                       |
| P1       | Testnet fault API                                                 | Retain and broadcast Core Node-side old commitments only on testnet                                 |
| P1       | ORDX channel drill                                                | Complete ORDX splicing, unlock, and lock with `bindingSat` rules                                    |
| P2       | Video capture tooling                                             | Capture PWA, L1 explorer, L2 explorer, indexer JSON, and skill output                               |
| P2       | Skill distribution                                                | Publish installable SAT20 Agent Wallet skill with stable docs links                                 |

P0 removes reliance on backend logs. P1 makes the Agent capable of proving exit and punishment. P2 turns the real testnet drill into reproducible product material.

## Current Testnet Baseline

The current testnet drill has verified:

| Capability                                   | Status                                                                                                                                              |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Normal client opens a channel with Core Node | Verified; normal users do not stake                                                                                                                 |
| Channel recovery / rebuild / expand          | Verified, including interrupted splicing-in recovery                                                                                                |
| sats unlock / lock                           | Verified                                                                                                                                            |
| Runes splicing-in/out and unlock/lock        | Verified                                                                                                                                            |
| BRC20 splicing-in/out and unlock/lock        | Verified                                                                                                                                            |
| Unknown-result convergence                   | Verified through visible transaction and state convergence                                                                                          |
| `stp.safety_snapshot`                        | Returns `READY_SAFE` in the validated flow                                                                                                          |
| Punishment coverage                          | Returns `COVERED` in the validated flow                                                                                                             |
| Real punish drill                            | Verified with old Core Node commitment broadcast and user-side punish package                                                                       |
| ORDX small-amount drill                      | Verified; `ordx:o` objects are ignored by STP ascend                                                                                                |
| Full PWA transaction polling                 | Still pending                                                                                                                                       |
| Short video                                  | Produced. See the [narrated demo](https://github.com/sat20-labs/docs-en/blob/main/ai/sat20-agent-wallet/video/sat20-agent-wallet-one-minute-en.mp4) |

## User-Facing Statement

The Agent should explain:

1. Your assets are not custodied by SatoshiNet.
2. STP channels preserve Bitcoin L1 commitment exits.
3. You can close or force close.
4. Assets moving freely on SatoshiNet can be locked or lock-with-expanded back into channel protection.
5. Every cross-layer action has txids, state, and verifiable chain evidence.

This is the central claim STP and SatoshiNet must prove: Bitcoin-native assets gain higher-efficiency liquidity while preserving user control under a Bitcoin-like safety model.
