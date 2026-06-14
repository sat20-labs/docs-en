# Asset Safety Model

SatoshiNet's safety goal is simple: users should always know where their assets are and how to recover them if a Core Node goes offline, fails, or behaves maliciously.

This goal is mainly achieved by STP.

## Safety Is Not a Balance Display

A wallet balance is not an asset safety proof. For an STP channel, an Agent or client needs to verify:

1. Whether the Bitcoin L1 channel point exists.
2. Whether the latest commitment transaction is usable.
3. Whether the commitment height moves monotonically.
4. Whether user and Core Node balances match the commitment state.
5. Whether revoked old states have punishment transaction coverage.
6. Whether force-close and post-CSV sweep paths can be constructed.

If this evidence is missing, the client should stop normal value movement instead of trusting Core Node status.

## Three Exit Capabilities

| Capability | Role |
| --- | --- |
| Cooperative close | Friendly close using the latest state when both sides are online |
| Force close | User exits with the latest commitment transaction when the peer is offline |
| Punish old state | User punishes a peer that broadcasts an old commitment transaction |

This mechanism follows the same RSMC idea used by the Lightning Network. STP extends it to more Bitcoin-native assets and SatoshiNet cross-layer state.

## Agent Role

An AI Agent should not custody private keys for users. The Agent should:

1. Call a wallet adapter.
2. Check safety snapshots.
3. Read commitment transactions and punish coverage.
4. Decide whether unlock, lock, splicing, or close can be executed.
5. Explain risks and next steps to the user.

Real signatures, private keys, mnemonic phrases, and channel databases should stay inside the wallet security boundary. SAT20 PWA Wallet is the recommended boundary.
