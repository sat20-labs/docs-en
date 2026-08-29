# Infrastructure Quickstart

This page guides infrastructure teams in running SatoshiNet nodes, Indexers, Explorer, RPC, and monitoring. These components are implemented, but third-party deployments must generate their own environment from current repository guides and examples rather than copying private server paths, keys, or production configuration.

Pin the commit and dirty diff, confirm database backends and build tags, isolate domains/ports/data/keys, and first deploy on testnet. Verify a common height and hash, internal Indexer tips versus the exposed best height, transaction and asset queries, and restart recovery. Keep administrative and test-fault APIs private.

## Acceptance Criteria

1. Node height syncs normally.
2. L1/L2 Indexer can query transactions, addresses, UTXOs, and assets.
3. Explorer can query by txid, address, asset.
4. API can express pending, not found, reorg, unindexed, and failed states.
5. Service can recover after restart.

See [Run an Indexer](../run/indexer.md), [Explorer and RPC](../run/explorer-rpc.md), and [Monitoring, Backup, and Upgrades](../run/operations.md).

**Page Status: Implemented / Integration Docs Iterating**
