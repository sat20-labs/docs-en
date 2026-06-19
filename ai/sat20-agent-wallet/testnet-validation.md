# SAT20 Agent Wallet Testnet Validation Record

This record documents what an AI Agent can observe and verify through the `sat20-agent-wallet` skill on Bitcoin testnet4 and SatoshiNet testnet. It keeps only evidence that a user, Agent, wallet adapter, L1/L2 explorer, or indexer can independently verify. It does not record debug logs, internal functions, temporary fixes, or runtime troubleshooting.

## Validation Principle

The Agent validates facts, not service claims.

| Evidence | What the Agent verifies |
| --- | --- |
| Bitcoin L1 tx / outpoint | funding, splicing, commitment, and punish visibility or confirmation |
| SatoshiNet tx / UTXO | anchor, unlock, lock, deAnchor, and L2 asset state |
| channel id / channel point | commitment input matches channel state |
| commit height | channel state advances monotonically |
| commitment export | user holds a broadcastable latest local commitment |
| punish coverage | revoked peer commitments have verified / broadcastable punish tx |
| reservation / transaction status | unfinished operations are tracked and not duplicated |

Only complete evidence lets the Agent mark a channel `READY_SAFE`.

## Verified Flow

The testnet drill has covered:

1. Normal client open with a Core Node, without staking.
2. Sats unlock and lock to advance commitment height.
3. Runes splicing-in, unlock, lock, and splicing-out.
4. BRC20 splicing-in, pending-to-spendable observation, unlock, lock, and splicing-out.
5. ORDX small-amount splicing-in, unlock, and lock, with `bindingSat` rules and `ordx:o` ignored during ascend.
6. Channel recovery with expand for assets already at the channel address.
7. Unknown network result handling by tracking txids/reservations rather than repeating value movement.
8. Testnet old Core Node commitment retention and broadcast.
9. Client-side punishment package construction and broadcast.
10. Fresh channel close-out after punishment.

## Representative Evidence

The validated material includes real Bitcoin testnet4 and SatoshiNet txids. The latest complete punish-oriented line uses a fresh channel, multi-asset movement, old Core Node commitment, and four confirmed punishment transactions.

Latest minimal safety loop, verified on 2026-06-18:

| Segment | Evidence |
| --- | --- |
| Normal open | `cda894102cfe72667c54e14b259b95d38037f8cbb6e39d1adeef7f8efb5d00f7:0`, confirmed at BTC testnet4 height `140402` |
| Retained Core Node commitment | `0d033af198e477de429c7ab656afec6e2cac5e672e62692fecf5b2a385c08f83`, retained at commit height `0` |
| State advance | sats unlock `d7a593b9301ecf795fc58d9c339b741663723e226cb984fb981a2ceebb953a11`, commit height `0 -> 1` |
| Old commitment broadcast | `0d033af198e477de429c7ab656afec6e2cac5e672e62692fecf5b2a385c08f83`, confirmed at BTC testnet4 height `140407` |
| Punish tx | `6221d7793f2bbc2687d959aa2c144e25cc01be2decf9dbf2e4069cd9d21376e8`, confirmed at BTC testnet4 height `140407` |
| Post-punish status | local client `status=-2`, Core Node `status=-1` |

This minimal loop proves the Agent-facing safety path: the PWA wallet holds the mnemonic and signs protocol messages; the Agent submits verifiable STP operations; after a revoked Core Node commitment appears on Bitcoin L1, the client wallet can build, broadcast, and verify punishment on L1. The canonical JSON evidence lives in [`sat20-labs/docs`](https://github.com/sat20-labs/docs/blob/main/ai/sat20-agent-wallet/evidence/testnet-minimal-punish-evidence-2026-06-18.json).

Current compact safety loop, also verified on 2026-06-18:

| Segment | Evidence |
| --- | --- |
| Fresh open funding | `dbaff278b630215bfe12543ab0f84287c7a3df424fc8381121a2057966003df1:0`, confirmed at BTC testnet4 height `140388` |
| BRC20 expand | `97e9fae9442559ac597649c03ad4eee095f6f58d4c2b5217fa6d7290b6747265`, commit height `1` |
| Sats unlock / lock | `4b3a2ab3a957f99884cb2e862b9879f61d8d9d2dcc1d1b79fd34eb77f4f45b04`, `41170da95b97d33723c47af6b1633e1d409b364aa01b9c0785595a44dbe5c19d` |
| BRC20 splicing-out | `ddf3e3e712d9b5bb343fccdf6c4485accb4587dcb6afb1731fe8a87c36b64238`, confirmed at BTC testnet4 height `140395` |
| Retained old Core Node-side commitment | `d6084d8f789a8977c5e6ed6f7245587aab71129df4776bf5b60a851cafda2939`, retained at commit height `4` |
| State advance after retain | `030b357ad599d900564831cef63f79a2e07531022c3af4ceed8dd0d0df37bfdc`, commit height `4 -> 5` |
| Punish package | `2364b55db774a2a032ec16f991b49ddc70ee7e4aa3b8c96aaf99960669c8ab7a`, `79a1fc6d17befb774aeee54e7f1675aa1f323653714441d1034ef1ead7912a22`, `f2d01ac1e9b1df247b45e2e4536efef8cfbfe70d563972197e7fa6f37f004e0c`, `4a426d996035e29e78ffb9f5fa570f0cce4ce26c17e1e39f06ed66303c7f7539` |

These txids are current testnet evidence. Older rollback-era L2 txids are intentionally omitted from this page because they are no longer independently queryable on the current SatoshiNet testnet indexer.

## What the Agent Learns

The drill teaches an Agent to:

1. Wait for L1 confirmation and L2 spendability instead of trusting commitment balance alone.
2. Use expand when assets are already at the channel address but not covered by the current commitment.
3. Treat unknown network results as pending evidence, not immediate failure.
4. Read safety snapshots before every value movement.
5. Recognize that a Core Node-side old commitment can be punished.
6. Stop using a punished/closed old channel and reopen a fresh safe channel when needed.

## Evidence Still Needed for Productized Use

1. PWA adapter should return standardized explorer and indexer URLs for every txid.
2. `stp.transaction` should include reservation, visible txids, confirmation count, channel id, error code, and next recommended action.
3. `stp.safety_snapshot` should be available directly from PWA without backend-log inspection.
4. Video capture should use PWA, L1 explorer, L2 explorer, indexer JSON, and skill output instead of terminal-only evidence.
