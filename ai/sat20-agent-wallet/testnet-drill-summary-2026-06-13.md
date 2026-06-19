# Testnet Drill Summary: 2026-06-13

This page keeps the Agent safety lessons from the 2026-06-13 testnet drill. Because the SatoshiNet testnet was later rolled back, old L2 txids and old evidence files are no longer listed as current public evidence. Current verifiable txids live in `testnet-validation.md` and in the canonical JSON evidence under `sat20-labs/docs`.

## Purpose

The drill demonstrated that an Agent-controlled SAT20 wallet can:

1. Open a client-Core STP channel without staking.
2. Bring Bitcoin-native assets into the channel.
3. Move assets between the channel and a SatoshiNet personal address.
4. Distinguish pending L2 assets from spendable L2 assets.
5. Retain and later observe an old Core Node-side commitment.
6. Build and broadcast punishment transactions.
7. Stop using the punished channel and return to a fresh safe channel.

## High-Level Timeline

1. Open a fresh channel and verify `READY_SAFE`.
2. Expand existing channel-address assets where needed.
3. Splicing-in Runes and BRC20 assets.
4. Wait for BRC20 anchor to become spendable before unlock.
5. Unlock and lock assets to advance the channel state.
6. Retain a Core Node-side commitment, then advance the state so the retained commitment becomes old.
7. Broadcast the old commitment on testnet.
8. Build and broadcast the punishment package.
9. Open a fresh channel after the punished channel is closed.
10. Verify the new channel has current commitment and punishment coverage.

## Agent Safety Conclusions

1. Ordinary users can connect to a Core Node without staking.
2. STP can handle multiple Bitcoin-native asset classes through channel operations.
3. Commitment balance and L2 spendable balance are different, and a safe Agent waits for spendability.
4. The user wallet can hold enough material to punish a revoked Core Node-side commitment.
5. After punishment, the Agent treats the old channel as closed and creates a fresh safety boundary.

## What the Public Demo Should Show

The public demo should show PWA wallet, Bitcoin testnet4 explorer, SatoshiNet explorer, indexer evidence, and skill output. It should not show internal server logs, temporary repair commands, or debugging details.

The key message is simple: STP lets Bitcoin-native assets move through SatoshiNet while preserving user-controlled exit and punishment paths.
