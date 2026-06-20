# STP Message Sequences

This document describes STP message order, on-chain results, state transitions, and Agent validation points by operation. It complements [STP Messages and Data Model](messages-and-data-model.md) and is intended for third-party client implementation, testnet drills, and AI Agent automated verification.

## General Flow Rules

Every STP operation that changes asset ownership follows these rules:

1. Read a safety snapshot before the operation and confirm the channel is operable.
2. The client internally selects or constructs inputs. A normal Agent does not directly provide asset UTXOs or fee UTXOs.
3. Both sides construct the new commitment state before revoking the old commitment state.
4. Before entering broadcast or final ack, the client must persist the pending operation, related txids, commitment transactions, and revocation material.
5. Commit height can only increase monotonically.
6. Network errors after broadcast are unknown results and must be recovered through L1/L2 indexers and peer state.
7. After the operation, read a safety snapshot again and verify punish coverage and asset facts.

## Open Channel

Goal: establish a private STP channel between Client Wallet and Core Node.

| Step | Message / Action | Validation Point |
| --- | --- | --- |
| 1 | Client Wallet discovers Core Node | Verify network, Core Node public key, capability list, CSV, and fees |
| 2 | `ChannelOpenReq` | Request a normal client-core private channel; ordinary users do not stake assets |
| 3 | `ChannelOpenResp` | Verify Core Node signature, channel parameters, and initial revocation / commitment point |
| 4 | Build Bitcoin L1 funding | Funding output must pay to the 2-of-2 channel address |
| 5 | `FundingCreatedReq` / `FundingCreatedResp` | Both sides exchange initial commitment, de-anchor, and related signatures |
| 6 | Broadcast funding | On unknown result, lock inputs and poll L1 |
| 7 | `FundingBroadcastedReq` / `FundingBroadcastedResp` | Both sides enter confirmation and anchor tracking |
| 8 | L1 funding confirmed; L2 ascend / anchor confirmed | L1/L2 indexers can verify both |
| 9 | Channel ready | Safety snapshot returns latest commitment and punish coverage |

Agent validation: the channel address is derived from client pubkey and Core Node pubkey; funding outpoint, initial commitment, L2 anchor, and commit height match.

## Splicing-In

Goal: add new Bitcoin L1 assets into an existing channel.

| Step | Message / Action | Validation Point |
| --- | --- | --- |
| 1 | Read safety snapshot | Channel ready, no pending operation, punish coverage complete |
| 2 | Adapter selects or constructs L1 asset input | Agent only provides asset, amount, and authorization; it does not choose UTXOs |
| 3 | `SplicingInReq` | Declares asset, amount, current commit height, and whether a splicing tx must be sent |
| 4 | `SplicingInResp` | Verify new capacity, new balances, service fee, and next revocation material |
| 5 | `SplicingInCommitSigReq` / `SplicingInCommitSigResp` | Exchange splicing, anchor, and commitment signatures |
| 6 | Broadcast related L1/L2 transactions | BRC20 may require transfer inscription and a transaction package |
| 7 | `SplicingInRevokeAndAckReq` / `SplicingInRevokeAndAckResp` | Revoke old state and confirm new commitment state |
| 8 | Wait for indexer convergence | L1 funding and L2 ascend / anchor can be verified |

Agent validation: the new asset enters commitment balance. If the indexer has not returned a spendable UTXO yet, mark it pending instead of continuing unlock/lock.

## Expand

Goal: include assets already at the channel address but not covered by the current commitment state.

Expand is common when:

1. The user has transferred assets to the channel address.
2. A previous splicing-in was interrupted by a network error.
3. Recovery found client-owned assets at the channel address.

The flow is similar to splicing-in, but the client must first determine through L1/L2 indexers whether the asset has already ascended. Already-ascended assets must not be anchored again. Only new assets at the channel address that can be proven unascended should enter the anchor path.

Agent validation: after expand, commit height increases, the asset is covered by the current commitment, and L2 supply does not increase through duplicate anchor.

## Unlock

Goal: release user-owned channel assets to the user's L2 address.

| Step | Message / Action | Validation Point |
| --- | --- | --- |
| 1 | Read safety snapshot | Channel ready, asset in commitment balance, punish coverage complete |
| 2 | `UnlockReq` | Request asset release to the user's L2 address |
| 3 | `UnlockResp` | Core Node accepts and returns revocation / next-revocation material |
| 4 | `UnlockCommitSigReq` / `UnlockCommitSigResp` | Exchange new commitment signatures and old-state revoke-and-ack |
| 5 | `UnlockRevokeAndAckReq` / `UnlockRevokeAndAckResp` | Complete old-state revocation and unlock transaction signatures |
| 6 | L2 indexer confirms | User L2 address receives assets and commit height increases monotonically |

Unlock does not require a user-supplied Bitcoin L1 fee rate or fee UTXO. SatoshiNet L2 can have zero-sat asset UTXOs.

Agent validation: user L2 spendable balance increases; channel commitment balance decreases; latest local and remote commitments are both updated.

## Lock

Goal: bring assets from the user's L2 address back under channel protection.

| Step | Message / Action | Validation Point |
| --- | --- | --- |
| 1 | Query L2 spendable UTXOs | The asset must be spendable, not merely pending |
| 2 | `LockReq` | Request locking the asset back into the channel |
| 3 | `LockResp` | Returns new commitment-signature material and next revocation material |
| 4 | `LockCommitSigAndRevokeReq` / `LockCommitSigAndRevokeResp` | Exchange signatures and revoke old state |
| 5 | `LockAckReq` / `LockAckResp` | Complete the state transition |
| 6 | Read safety snapshot | Commit height increases and the asset returns to channel control |

Agent validation: the asset moves from user L2 spendable balance to channel commitment balance, and the old remote commitment has punish coverage.

## Lock-With-Expand

Goal: bring user assets back under channel control when channel capacity or asset coverage is insufficient.

Lock-with-expand is an asset-safety capability, not a simple transfer API. It lets users bring assets already on L2 or near the channel address back into commitment protection.

Agent validation: after the operation, the asset is covered by the current commitment. If expand is needed, it must not duplicate ascend. If channel state is incomplete, recovery should run first.

## Splicing-Out

Goal: exit channel assets to a Bitcoin L1 address.

| Step | Message / Action | Validation Point |
| --- | --- | --- |
| 1 | Read safety snapshot | Channel ready, no pending operation, asset in commitment balance |
| 2 | Adapter builds exit transaction package | BRC20 may need transfer inscription; Runes/ORDX follow L1 protocol rules |
| 3 | `SplicingOutReq` | Request asset exit to an L1 address |
| 4 | `SplicingOutResp` | Verify service fee, new capacity, new balances, and next revocation material |
| 5 | `SplicingOutCommitSigReq` / `SplicingOutCommitSigResp` | Exchange L1 exit, L2 descend, and commitment signatures |
| 6 | Broadcast L2 descend / de-anchor and related L1 transactions | Unknown result enters recovery; inputs are not double-spent |
| 7 | `SplicingOutRevokeAndAckReq` / `SplicingOutRevokeAndAckResp` | Revoke old state and complete the update |
| 8 | L1/L2 indexers confirm | L2 asset decreases and L1 destination receives the asset |

Agent validation: L2 descend and L1 output can be linked; asset, amount, and destination match user authorization.

## Cooperative Close

Goal: close the channel cooperatively and exit assets according to the latest state.

| Step | Message / Action | Validation Point |
| --- | --- | --- |
| 1 | Read safety snapshot | Channel ready, no pending operation, latest commitment verifiable |
| 2 | `ChannelCloseReq` | Request cooperative close |
| 3 | `ChannelCloseResp` | Core Node returns closing, de-anchor, and related transaction signatures |
| 4 | `ClosingSignedReq` / `ClosingSignedResp` | Client signs and confirms close transaction |
| 5 | Broadcast close-related transactions | On unknown result, query L1/L2 tx visibility |
| 6 | `ClosingBroadcastedReq` / `ClosingBroadcastedResp` | Both sides enter closed or waiting-confirmation state |
| 7 | L1/L2 indexers confirm | Channel assets exit or enter user-controlled addresses |

Agent validation: asset ownership before and after close is consistent, and no user asset remains unexpectedly at the channel address.

## Force Close and Sweep

Goal: let the user close unilaterally when the Core Node is offline, refuses service, or cannot cooperate.

| Step | Action | Validation Point |
| --- | --- | --- |
| 1 | `stp.force_close_plan` | Returns latest local commitment, CSV delay, and later sweep conditions |
| 2 | Broadcast local commitment | Must be the latest commitment state |
| 3 | Wait for CSV or other spend condition | Monitor whether peer broadcasts an old remote commitment |
| 4 | `stp.sweep_build` | Build sweep transaction, dry-run by default |
| 5 | Broadcast sweep after user authorization | User recovers sweepable assets |

If the Core Node broadcasts its held commitment, the user can choose to reopen the channel or sweep user-owned channel assets. The user enters punish only when the Core Node broadcasts an old commitment.

## Punish

Goal: when the Core Node broadcasts an old remote commitment, the user uses revocation material to punish the old state.

| Step | Action | Validation Point |
| --- | --- | --- |
| 1 | Watchtower or Agent detects old commitment | txid matches a revoked remote commitment |
| 2 | `stp.punish_status` | Confirms punish material exists and CSV window is still open |
| 3 | `stp.punish_build` | Builds and dry-run verifies punish tx |
| 4 | User authorization or testnet drill authorization | Mainnet must protect the user authorization boundary |
| 5 | `stp.punish_broadcast` | Broadcasts punish transaction |
| 6 | L1 indexer confirms | Channel enters punished / closed state |

Testnet can drill this process through old-commitment retention and old-commitment broadcast interfaces. Mainnet must not expose fault-injection interfaces.

## Reopen / Rebuild / Restore

Goal: recover user asset control after channel close, state loss, or unknown chain result.

| Capability | Flow Focus | Validation Point |
| --- | --- | --- |
| Restore | Recover from local backup, peer state, or persisted data | Latest commitment and punish coverage can be proven |
| Reopen | Old channel is closed but channel address still holds user assets | User may need to add new funding; unnecessary duplicate fees are avoided |
| Rebuild | Channel point or state was lost and must be rebuilt from L1/L2 ledger | Assets are assigned from ascend / descend records without duplicate anchor |
| Expand | Assets at the channel address are not covered by current commitment | Assets enter commitment state and commit height increases |

Agent validation: recovery must regenerate a safety snapshot. If `PUNISH_COVERAGE_UNKNOWN`, unclear pending operation, asset-ledger mismatch, or unknown channel state remains, normal value movement must not continue.

## Agent Drill Sequence

A complete testnet drill can follow this order:

1. Install SAT20 Wallet or an equivalent secure wallet boundary.
2. Create a test wallet and connect to the default Core Node.
3. Open a normal client-core channel.
4. Unlock sats and observe commit height increase.
5. Lock sats and confirm assets return to channel protection.
6. Splicing-in Runes and wait for L1/L2 indexer confirmation.
7. Unlock / lock Runes.
8. Splicing-in BRC20 and verify the transfer-inscription transaction package.
9. Unlock / lock BRC20.
10. Splicing-out at least one protocol asset to L1.
11. Export safety snapshot, commitment export, and punish status.
12. Trigger old-commitment broadcast on testnet, then build and broadcast punish tx.
13. Explain every transaction through L1 explorer, L2 explorer, L1 indexer, and L2 indexer.

The drill is not about showing balance changes. It proves that the user holds the latest exit path, can detect old states, can punish a malicious peer, and can independently verify asset facts through indexers.
