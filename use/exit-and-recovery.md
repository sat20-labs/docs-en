# Exit SatoshiNet and Recover

Exit and recovery are different operations. Normal exit prefers splicing-out or cooperative close; force close and punishment are abnormal protection paths.

## Normal Exit

1. Return a subset to Bitcoin L1 with splicing-out, verifying destination, asset, amount, and fee.
2. Lock assets from an L2 user address back into the channel first; insufficient capacity requires explicit paid-expansion approval.
3. Use cooperative close to end the whole channel, and wait for completion before opening another channel.
4. Record L2 deAnchor/channel actions and the final L1 transaction, then verify on both networks.

## Abnormal Protection

- Export a safety snapshot and latest commitment before changing local data when the Core Node is unavailable or states diverge.
- Force close may cost more or take longer. Revoked commitments enter the punishment path; test fault injection must be unavailable on mainnet.
- Treat unknown network results as possibly successful. Inspect reservations, logs, L1/L2 transactions, and server state first.

## Wallet and Account Recovery

1. A root-account mnemonic or recovery package restores the root identity and DKVS-managed account data.
2. A non-root wallet mnemonic restores only that wallet's chain data.
3. Channel recovery depends on the correct subaccount, server ping snapshot, commit height, and local safety material. Monitor wallets do not recover channels.
4. Old and restored profiles must not write the same wallet/channel concurrently.
5. Validate RGB11 recovery separately for accounts with and without RGB11 data.

**Page Status: Implemented / Abnormal Paths Continue Under Validation**
