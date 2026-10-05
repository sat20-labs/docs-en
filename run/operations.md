# Monitoring, Backup, and Upgrades

Node and infrastructure operators need repeatable monitoring, backup, upgrade, and recovery processes.

## Minimum Monitoring Surface

1. Node height, peers, mempool, and block production state.
2. L1/L2 indexer height and reorg state.
3. STP channel and contract state.
4. Data directories, wallet directories, and key database backups.
5. Version upgrades, rollbacks, and compatibility checks.
6. Alerts, logs, and public status pages.

Monitor both process liveness and business progress: a live process with stalled blocks, a lagging Indexer tip, or non-converging channel commits must alert.

Backups must bind chain/Indexer data to a network, tip hash/height, database backend, and code version. STP wallet/channel state, commitments, watchtower data, and reservations are not disposable chain snapshots. Back up the complete DKVS instance DB to keep EndpointID, current records, collection metadata, and each key's latest generation index consistent. SDK confirmed replicas, synchronization metadata, ACKs, and original-request outbox entries also require a consistent backup. The current implementation has no change log or historical cursor head; restoring an old backup does not permit bypassing source / baseline checks or marking it READY. See [DKVS](../protocol/dkvs/readme.md).

For upgrades, freeze the commit and dirty diff, hash artifacts, run unit/virtual-network/E2E tests sequentially where required, and validate on testnet before staged rollout. Already broadcast actions must remain monitored and cannot become failed merely because of a restart or a 120-second timeout. PWA upgrades should ask users to close and reopen instead of immediately taking over active old pages. SatoshiNet rollback is testnet-only and mainnet must reject it.

Unknown network results are possibly successful; do not automatically rebroadcast, delete reservations, or delete contracts. Diagnose channel divergence from the first common commit/invoke point and record every recovery action.

**Page Status: Operational Baseline / Iterating**
