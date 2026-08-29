# L1 / L2 Indexer

Indexers are SatoshiNet's asset fact layer. L1 Indexer handles Bitcoin mainnet multi-protocol asset facts. L2 Indexer handles SatoshiNet transactions, UTXOs, channels, contracts, and cross-layer states.

## Operating Value

1. Provide asset queries for wallets and exchanges.
2. Provide visualized data for explorers.
3. Provide cross-layer evidence for STP and Agents.
4. Provide state queries for DEX, DAO, and contract applications.
5. Reduce single-point dependency through multi-party operation.

## Current Implementation Boundary

| Area | Current implementation |
| --- | --- |
| L1 entry | Root `main.go` in the `indexer` repository; `cmd/main.go` is a tool/test entry |
| L1 source | Network-specific bitcoind RPC; mainnet and testnet/testnet4 configuration and databases must be isolated |
| L1 database | Pebble by default; Badger only under an explicit build tag. A Badger testnet deployment does not switch the SDK, STP, or every node |
| L1 protocols | Base UTXO/sat ranges, Ordinals, ORDX, BRC-20, Runes, rare sats, and related protocols |
| L1 reorg | Runtime state plus a delayed persistence window, followed by rollback and replay; backups must record both chain tip and database height |
| L2 Indexer | `satoshinet/indexer`, running with SatoshiNet nodes for L2 UTXOs, ascend/descend, channels, contracts, and DKVS |
| Public APIs | L1 and L2 are exposed through configured proxies; callers must specify the network rather than guess from a txid |

Before build and startup, check the current `build.sh`, environment configuration, and build tags. Record the commit, dirty diff, Go version, tags, and binary SHA-256; then compare chain tip, internal Indexer tip, public best height, and block hash. Reorg testing must prove that invalid blocks do not permanently pollute state.

Cross-verification compares network, tip/hash, outpoint, protocol, amount, confirmation state, and error semantics. Same-height/different-hash or same-outpoint/different-asset results are data divergence, not an API success to be hidden.

See the [API source map](../build/api-source-map.md).

**Page Status: Implemented / Iterating**
