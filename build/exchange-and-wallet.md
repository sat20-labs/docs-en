# Exchange and Wallet Integration

Exchanges and wallets are important entry points for the SatoshiNet ecosystem. They need more than transfer APIs; they need to explain whether an asset is on Bitcoin L1, in a channel, in a personal SatoshiNet address, in a contract, or pending.

For exchanges and wallets, the indexer is the asset fact layer. Deposits, withdrawals, splicing, unlock, lock, balance display, and risk control must be based on L1/L2 indexer evidence: txid, vout, asset protocol, confirmations, and channel state.

## Integration Goals

| Role | Goal |
| --- | --- |
| Wallet | Manage private keys, authorization, signatures, channel state, and user safety prompts |
| Exchange | Support deposits, withdrawals, asset identification, confirmations, and abnormal-state handling |
| Indexer | Provide L1/L2 assets, transactions, UTXOs, channels, and contract state |
| Agent | Read evidence, explain risk, and call wallet adapters |

## What Wallets Must Protect

Wallets must protect:

1. Mnemonic phrases and private keys.
2. Channel database.
3. Latest commitment transactions.
4. Punishment material for revoked states.
5. Pending reservations.
6. User authorization records.

Agents do not directly access these secrets.

## What Exchanges Must Understand

When integrating SatoshiNet, exchanges distinguish:

1. Bitcoin L1 confirmation.
2. SatoshiNet L2 confirmation.
3. STP channel state.
4. Anchor / deAnchor state.
5. Whether assets are spendable.
6. Whether pending operations exist.

Looking only at balances can lead to wrong decisions during cross-layer operations.

Looking only at a single indexer response, without txid, vout, confirmations, reorg, mempool, and cross-layer correspondence, can also lead to wrong decisions. Critical asset movement should keep verifiable evidence.

## Recommended Interfaces

1. L1 / L2 address summary.
2. Transaction details.
3. UTXO asset details.
4. Channel ledger.
5. Anchor / deAnchor records.
6. STP transaction / reservation query.
7. Safety snapshot.
8. Reorg / mempool / not indexed state.

Future exchange documentation can further split into deposits, withdrawals, risk control, reconciliation, and abnormal recovery.
