# Verify Transactions and Assets with Explorer

Explorer makes chain evidence readable but does not replace wallet authorization, client validation, or Indexer APIs.

## Current Entries

| Network | Entry | Use |
| --- | --- | --- |
| Bitcoin mainnet | [mempool.space](https://mempool.space/) | L1 transactions, addresses, UTXOs, and confirmations |
| Bitcoin testnet4 | [mempool.space/testnet4](https://mempool.space/testnet4/) | Test L1 transactions and confirmations |
| SatoshiNet mainnet | [mempool.sat20.org](https://mempool.sat20.org/) | SatoshiNet mainnet transactions and addresses |
| SatoshiNet mainnet app explorer | [mainnet.sat20.org/browser/app](https://mainnet.sat20.org/browser/app/) | Asset, UTXO, and application views |
| SatoshiNet testnet app explorer | [testnet.sat20.org/browser/app](https://testnet.sat20.org/browser/app/) | Testnet asset, UTXO, and application views |

## Standard Verification

1. Copy txid and identify Bitcoin versus SatoshiNet explicitly.
2. Confirm height, confirmations, inputs, outputs, assets, and spend state in the matching Explorer.
3. Verify asset and protocol details through the matching L1/L2 Indexer.
4. For STP, connect L1 evidence to anchor/deAnchor, commit height, and L2 state.
5. For contracts, verify call transaction, invoke item, Result TX, and contract state.

For `not found`, first verify network, txid, and Indexer height. A confirmed transaction with a pending contract item is no longer “waiting for L1 confirmation.” Reorg can invalidate old L2 evidence, so historical records include network, height, and verification date.

**Page Status: Implemented / Display Iterating; entries checked on 2026-08-29**
