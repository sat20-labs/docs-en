# Testnet Drill Summary: 2026-06-13

This page summarizes the final testnet drill line used for SAT20 Agent Wallet documentation. It omits intermediate troubleshooting and focuses on what an Agent can learn and verify.

## Purpose

The drill demonstrates that an Agent-controlled SAT20 wallet can:

1. Open a client-Core STP channel.
2. Bring Bitcoin-native assets into the channel.
3. Move assets between the channel and SatoshiNet personal address.
4. Distinguish pending L2 assets from spendable L2 assets.
5. Retain and later observe an old Core Node-side commitment.
6. Build and broadcast punishment transactions.
7. Stop using the punished channel and return to a fresh safe channel.

## High-Level Timeline

1. Open a fresh channel and verify `READY_SAFE`.
2. Expand existing channel-address assets where needed.
3. Splicing-in Runes and BRC20 assets.
4. Wait for BRC20 anchor to become spendable before unlock.
5. Unlock and lock BRC20 to advance the channel state.
6. Retain a Core Node-side commitment, then advance the state so the retained commitment becomes old.
7. Broadcast the old commitment on testnet.
8. Build and broadcast the punishment package.
9. Open a fresh channel after the punished channel is closed.
10. Verify the new channel has current commitment and punishment coverage.

## Representative Txids

| Step | Evidence |
| --- | --- |
| Fresh open | `fc2282849c15286fe2e7dba05847dad376d4d49a04fde06111d0a3ceac48093c` |
| Runes splicing-in | `4ab20f45818dc73e11ad5834249f358a017c5c6b384074530c9e96e5364f06ed` |
| BRC20 splicing-in | `c6481a19471dc2fc15ec411f9c2b04f5c48a13ea33a74e4e4be494cc3f703793` |
| BRC20 unlock / lock | `2e5e0fe1f44a8a6e1754558c5a9829a2b78bd6b661c7ba10f215e755888cd862`, `63fd7bf150e2678956fa765b4780c2ff5cf07cd203d0d04e7eb3ae31f7e3c289` |
| Old commitment | `151226853632d61d177b3279fcf99a7c144beaa3d018abd55f00f5b2adc24909` |
| Punish package | `459877810aa391f4da5ce6a7b0a817daee0dd63f4a24b8d77062dee7ef47fbee`, `23b3e6809eb5277ff9da7fb767ca8dc54e75f034104069f8c8a873b5e7c41451`, `c80dd18b79bbf1600a1a630be953de7e9f698d96c5b6a1d20eecf8f27dbb6859`, `22643d7c0b8f4a6aee93bcafceecc318ca56b9d31196e17b87fda43a9ee845d5` |
| Fresh open after punish | `e9c283cc9979bafa4f32f872dc2d0c910b89d810ef6801a1ad55bbeb5e585782` |

## Agent Safety Conclusions

The drill supports these conclusions:

1. Ordinary users can connect to a Core Node without staking.
2. STP can handle multiple Bitcoin-native asset classes through channel operations.
3. Commitment balance and L2 spendable balance are different, and a safe Agent waits for spendability.
4. The user wallet can hold enough material to punish a revoked Core Node-side commitment.
5. After punishment, the Agent treats the old channel as closed and creates a fresh safety boundary.

## What the Public Demo Should Show

The public demo should show PWA wallet, Bitcoin testnet4 explorer, SatoshiNet explorer, indexer evidence, and skill output. It should not show internal server logs, temporary repair commands, or debugging details.

The key message is simple: STP lets Bitcoin-native assets move through SatoshiNet while preserving user-controlled exit and punishment paths.
