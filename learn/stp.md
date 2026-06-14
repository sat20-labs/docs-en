# STP Introduction

STP stands for Satoshi Transcending Protocol. It defines how Bitcoin L1 assets enter SatoshiNet, move inside SatoshiNet, and exit back to Bitcoin L1.

The core of STP is not a "bridge"; it is a channel.

STP must be understood together with indexers. The indexer explains Bitcoin L1 asset facts: which UTXO carries which asset, whether it is confirmed, whether it is spent, and whether the protocol event is valid. STP builds channel control on top of those facts so users can enter SatoshiNet and exit in abnormal situations.

## What STP Does

| Action | Meaning |
| --- | --- |
| open | Open a channel between a user and a Core Node |
| splicing-in | Add Bitcoin L1 assets into an existing channel |
| unlock | Release channel assets to a personal SatoshiNet address |
| lock | Bring personal SatoshiNet assets back into the channel |
| lock-with-expand | Restore channel protection when channel capacity is insufficient |
| splicing-out | Exit channel assets back to Bitcoin L1 |
| close | Close the channel |
| punish | Punish an old commitment transaction |

These actions are not isolated transactions. They are protocol state machines involving Bitcoin L1 transactions, SatoshiNet transactions, commitment-state updates, signature exchange, and confirmations. Every cross-layer action needs L1/L2 indexer evidence to prove that asset facts and channel state are consistent.

## Ordinary Users Do Not Stake

An ordinary user who connects to a Core Node and opens a private channel does not need to pre-stake assets. Staking belongs to the path where a node connects to a Bootstrap Node and prepares to become a Core Node.

User channels and node-staking channels are different scenarios.

## Meaning for Agents

STP is well suited for AI Agents because it has clear states, transactions, evidence, and recovery paths. An Agent can turn a user goal into verifiable steps:

1. Query wallet and channel state.
2. Check the safety snapshot.
3. Choose open, splicing, unlock, lock, or close.
4. Poll L1/L2 transactions and asset states through indexers.
5. Enter recovery when a result is unknown.

The full protocol whitepaper is [STP Technical Whitepaper](../protocol/stp/readme.md).
