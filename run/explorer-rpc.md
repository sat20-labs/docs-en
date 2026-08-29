# Explorer / RPC

Explorer and RPC are public entry points for users, developers, wallets, exchanges, and Agents to verify SatoshiNet state.

## Required Capabilities

1. Query by txid, address, asset, and contract.
2. Show cross-layer evidence between Bitcoin L1 and SatoshiNet L2.
3. Show STP channels, commit height, ascend / descend, and punish evidence.
4. Show contract deployment, invocation, and Result TX.
5. Express pending, not found, reorg, unindexed, and failed states.

## Public Entries and Boundaries

- SatoshiNet mainnet transaction explorer: [mempool.sat20.org](https://mempool.sat20.org/).
- Mainnet application explorer: [mainnet.sat20.org/browser/app](https://mainnet.sat20.org/browser/app/).
- Testnet application explorer: [testnet.sat20.org/browser/app](https://testnet.sat20.org/browser/app/).
- Bitcoin L1 uses [mempool.space](https://mempool.space/) and [testnet4](https://mempool.space/testnet4/).

An Explorer is a presentation layer. A missing route or field does not prove that a transaction is absent; query the PWA-configured L1/L2 Indexer endpoint with an explicit network and API proxy.

Public RPC validation must compare best height/hash, transactions, addresses, UTXOs, assets, channels, and contracts. Mainnet and testnet must never silently fall back to each other. Distinguish `not found`, `pending`, `unindexed`, `reorged`, and unavailable states. Keep write, administrative, and fault-injection APIs private, and require Explorer link builders to receive an explicit Bitcoin-or-SatoshiNet identifier.

See [Verify Transactions and Assets with Explorers](../use/explorer-verification.md).

**Page Status: Implemented / Iterating**
