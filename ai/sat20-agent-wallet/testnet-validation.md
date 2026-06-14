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

Key evidence:

| Segment | Evidence |
| --- | --- |
| Fresh open funding | `fc2282849c15286fe2e7dba05847dad376d4d49a04fde06111d0a3ceac48093c` |
| Runes splicing-in | `4ab20f45818dc73e11ad5834249f358a017c5c6b384074530c9e96e5364f06ed` |
| BRC20 splicing-in package | commit `03fb93e84154bd55c4fb47c5f08eb839189229ea13f6ade710c9b2bd6ff24d03`, reveal `015b6d3cbfce0ef37899d812a68713cfbb89fd84e25616fda6edf648aa52b66f`, splicing `c6481a19471dc2fc15ec411f9c2b04f5c48a13ea33a74e4e4be494cc3f703793` |
| BRC20 unlock / lock | `2e5e0fe1f44a8a6e1754558c5a9829a2b78bd6b661c7ba10f215e755888cd862`, `63fd7bf150e2678956fa765b4780c2ff5cf07cd203d0d04e7eb3ae31f7e3c289` |
| Retained old Core Node-side commitment | `151226853632d61d177b3279fcf99a7c144beaa3d018abd55f00f5b2adc24909` |
| State advance after retain | `86310825ffa2d7d229d95b175c164ed2e32cefa354459f2060d68fb65ea326d7` |
| Punish package | `459877810aa391f4da5ce6a7b0a817daee0dd63f4a24b8d77062dee7ef47fbee`, `23b3e6809eb5277ff9da7fb767ca8dc54e75f034104069f8c8a873b5e7c41451`, `c80dd18b79bbf1600a1a630be953de7e9f698d96c5b6a1d20eecf8f27dbb6859`, `22643d7c0b8f4a6aee93bcafceecc318ca56b9d31196e17b87fda43a9ee845d5` |
| Fresh open after punish | `e9c283cc9979bafa4f32f872dc2d0c910b89d810ef6801a1ad55bbeb5e585782` |
| Fresh channel safety close-out | unlock `b88653f646e192765acaecbf338138c057c4ae420cbfddf7476f3cf5ea7994a8`, punish `84524f5aaa367f6ff7bcb15e1d01f8f7fa97fa93779c04cb96bc5ec58afa31a4` |

These txids are testnet evidence. They are suitable for technical demos and Agent verification examples, not for mainnet claims.

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
