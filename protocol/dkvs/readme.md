# DKVS: SatoshiNet Distributed Key-Value Storage

> Updated 2026-10-05. This page describes the DKVS data model, write authorization, synchronization protocol, and SDK integration boundaries.

DKVS is an owner-controlled small-data storage and synchronization layer for account management, encrypted backups, and application configuration. It provides atomic writes at one service node and eventual convergence of network-visible data, without cross-node linearizability or arbitrary multi-primary merging.

## Core Principles

1. Keys express stable business identity; schema upgrades do not create versioned keys.
2. `Seq` orders updates within one key's current lifetime; `ETag = RecordHash(record)` supports exact CAS.
3. Deletion physically removes current records. There are no persistent tombstones, delete floors, or mutation logs.
4. Generation belongs to the current endpoint and collection, supporting Wallet sync and write authorization; P2P does not propagate generation.
5. New Wallet writes validate CAS, current binding, request signatures, and this endpoint's prefix generations together.
6. `FREE_LOCAL` is an endpoint-local fee and retention policy, without cross-node backup guarantees.
7. SDK `dkvsManager` owns clients, subscriptions, replicas, outbox, and background synchronization; the PWA uses domain-level WASM APIs.
8. Unreleased functionality retains no legacy protocol compatibility, dual writes, or migration paths.

## Stable Keys and Authority

| Namespace | Purpose and authority | Placement / replication boundary |
| --- | --- | --- |
| `/account/<network>/<root-address>` | Root-signed account service binding | Network-replicated control record |
| `/personal/<account>/<module>/...` | Private domain data owned by the account | FREE_LOCAL or network storage |
| `/blob/<account>/<blob-key>` | Owner's encrypted larger objects | FREE_LOCAL or network storage |
| `/name/<did>` | Profile controlled by the current DID owner | Network storage |
| `/svc/<did>/...` | Service configuration controlled by the current DID owner | Network storage |
| `/mail/<account>/...` | Mailbox managed by MessageManager / owner | Currently bound CoreNode, AccountBound |
| `/topic/...` | Topic service state | Service policy / protocol-defined |
| `/sys/...` | System records authorized by SystemVerifier | Network storage |
| `/tmp/...` | Temporary data constrained by signer and local policy | LocalOnly |

The current root-signed AccountServiceDescriptor binding record is the binding fact, containing AccountID, CoreNodeID, and capabilities. Ordinary Wallet writes must be accepted by the currently bound CoreNode; standalone binding creation is a validated bootstrap exception.

A collection defines the scope of synchronization, endpoint-local generation, and current-set roots. Comparing Seq across keys or generation across endpoints does not establish network-wide freshness.

## Records, Deletion, and Recreation

A signed record contains `Version / Key / Value / Seq / IssueHeight / TTL / FeeProof / Flags / PubKey or account identity / Signature`.

- Initial creation uses `Seq = 1`; updates use current Seq + 1 and match the current ETag.
- A deletion command uses `FlagTombstone`, current Seq + 1, and a Value that signs the target record's RecordHash, satisfying exact CAS.
- Successful deletion removes the record, hash index, and the key's latest change-generation index. Snapshots contain no deletion commands.
- A deleted key is absent and recreation starts at `Seq = 1`; historical high sequences are not lifetime floors.
- With `TTL > 0`, expiry height is IssueHeight + TTL. Retention with `TTL = 0` is determined by paid policies and other applicable rules.
- New Wallet RPC writes require `currentHeight - 2 <= IssueHeight <= currentHeight`; this short window does not reject old but still valid current records in P2P.

Ordinary values, blobs, batches, pages, and complete replicas have separate capacity limits. DKVS stores bounded objects and references without promising general-purpose large-file storage.

The SDK Blob codec uses one format: `DKB1 + uint32(metadata length) + metadata + data`. The length field is big-endian, metadata may be empty, and data must be nonempty. The fixed eight-byte envelope counts toward the Blob value limit. Data bytes remain unchanged, without implicit compression. Decoding does not infer an unframed format or automatically migrate or rewrite existing raw values; the direct record API still returns the stored bytes.

## Wallet APIs and Write Authorization

Current application endpoints:

```text
GET  /v3/dkvs/config
GET  /v3/dkvs/record?key=...
GET  /v3/dkvs/key-state?key=...
POST /v3/dkvs/records/batch-cas
POST /v3/dkvs/active/sync
POST /v3/dkvs/active/watch
POST /v3/dkvs/prefixes/read
```

`/checkpoint`, `/snapshot`, and node `/subscriptions` are node-local administration endpoints, not a compatibility protocol for PWA managed replicas. There is no separate `/write-context` endpoint; the SDK obtains write context from completed synchronization metadata.

A write request includes RequestID, EndpointID, ordered mutations, each key's `ExpectAbsent` or `ExpectedETag`, and a wallet request signature over the endpoint, prefix generations, RequestID, record hashes, and CAS predicates. The record signature proves authorship; the request signature authorizes this specific write and cannot be replaced by copying a historical record.

The server verifies current binding, generation, exact CAS, Seq, signatures, permissions, fees, and quotas. Before committing, it confirms that payment and capacity validation still applies at the current local height and policy. Changed conditions require revalidation of the original authorized request. Failure of any mutation prevents partial submission; all changes commit atomically under one commit lock and DB batch.

After `STALE_GENERATION / WRITE_CONFLICT / INVALID_SEQUENCE`, the domain must synchronize current state and build a new business request. When the network outcome is unknown, the outbox may only replay the original complete signed request; it cannot change its generation, renew authorization, or replace CAS predicates. Definite record TTL expiry returns `DKVS_EXPIRED_RECORD`; the SDK discards the rejected expired outbox without changing the original TTL, signing height, or authorization.

## Active Sync / Watch and Confirmed Replicas

The SDK explicitly registers managed prefixes, persists them, and restores them on restart. Ordinary Get does not establish a permanent subscription. Subscribed data comes from the confirmed replica; unsubscribed data is fetched online on each read without a short-term read cache. On-demand and explicit authoritative reads share one five-second deadline covering the configuration, key-state, record, directory pages, and verification-height queries needed by that operation; steps do not reset it. Active sync likewise uses the caller's context for configuration and pages, and cancellation prevents result installation. Managed persistent replicas support offline reads of the last confirmed state.

An ActiveScope identifies a collection, optionally restricted to specific keys. Metadata contains `EndpointID / scope / generation / root / ViewHeight`.

- Full sync returns the current set, with pages pinned to one generation, root, and validation height. Changes to the server's current set invalidate the cursor and require a new read.
- Incremental sync returns records changed after the completed generation that still currently exist; it is not event history.
- Deletion returns no historical tombstone. The SDK verifies the current root after merging candidates and falls back to full sync when incremental results cannot explain a deletion.
- Only validated complete results atomically install records and synchronization metadata. Write ACKs do not directly advance completed sync positions.
- Local Wallet installation checks a concurrent-update baseline; outbox entries remain separate from confirmed records.
- One terminal uses one aggregated long-poll. Current defaults allow 16 prefixes, 256 records per page, and a 20-second Watch timeout. A response contains at most one changed scope's current page.
- The server maintains no persistent terminal session, offline event queue, or mutation log. Slow consumers read current state again.

The SDK stores confirmed records, managed prefixes, endpoint-local sync metadata, per-prefix ACK high watermarks, and original-request outbox entries. It stores no P2P source ownership, deletion history, or cross-node version table.

Sending and receiving return separate results. Send conflicts, expired original signing heights, funding pauses, and transient network failures do not block receiving sync / Watch that passes same-endpoint validation. Conflicting requests retain their original authorization and error while automatic submission stops. Transient network failures retry the original authorization at the existing five-second interval while Watch continues receiving. Receiving must still pass its own source and baseline checks. Definite storage downgrades, quota refusals, and structured request rejections return business errors, retaining the original request and rejection details while stopping automatic submission. Transient failures such as HTTP 429 retain original-request retries. The transport coordinator releases its slot and wakes waiters on return or panic; unrecoverable local errors retain fail-fast behavior.

Errors from account background tasks and mailbox maintenance remain visible and are returned to callers independently; they do not block managed receiving that passes its own validation. Failed tasks remain pending while other tasks continue, so a business refusal cannot block confirmed-change notifications. If a completed Watch page falls behind a concurrent foreground write ACK, the SDK discards the old page and reads current state through the existing bounded synchronization flow. The fresh response must still pass endpoint, ACK lower-bound, root, and installation-baseline checks.

Offline replicas represent only the last confirmed state and cannot prove current network freshness. Wall-clock time cannot fabricate a trusted chain height. Connecting to a different EndpointID does not automatically authorize replacement of an old replica. The current SDK blocks direct switching: active FREE_LOCAL records or their pending outbox cause a local-only endpoint mismatch; otherwise an explicit source / baseline decision is still required.

Refreshing the same fully synchronized directory set on the same endpoint preserves confirmed-read eligibility. After a network refresh failure, the OFFLINE_READY replica remains readable; a successful refresh restores READY. Initial synchronization, incomplete synchronization, and newly registered directories cannot gain offline-read eligibility through this path. Writes retain online authorization and CAS checks. A directory ViewHeight from full sync or Watch only advances the known endpoint verification height, so a delayed page cannot lower expiry checks for another directory. An explicit best-height query can still confirm an actual chain rollback.

Reads from an explicitly selected recovery node or another on-demand source validate that result at the source's current height without changing the managed replica's verification height. Before refreshing the shared verification height, a known EndpointID must match the persisted replica source. Reads such as storage estimates cannot combine another source's policy with the local replica height. A temporary online refresh failure on the same source may retain its known trusted height; network failure cannot waive source consistency. A source mismatch returns an endpoint mismatch while preserving the original replica and its height.

Get, directory listing, readiness checks, and write builders using local state share one confirmation boundary: every registered or pending directory has completed synchronization, and the client's known EndpointID matches the replica source. A known different source returns an endpoint mismatch while preserving the old replica. URL aliases with the same EndpointID may share the replica. Before a client confirms its source, it may read the last confirmed offline view, which remains attributed to the replica's persisted EndpointID rather than an unknown endpoint.

After the DKVS Manager is recreated, an eligible confirmed replica restores an unknown verification height from its persisted ViewHeight; zero is also a known height. An already observed online height takes precedence. Get and directory listing apply consistent signature and TTL checks. This offline boundary describes only the last confirmed view, not the latest chain height.

The account managed-data confirmation wait observes only locally confirmed state/blob records, checking signatures, TTL, profile content, and data revision/hash relationships. When not ready, it wakes the existing DKVS synchronization worker. The wait itself neither starts nor waits for synchronization HTTP. The caller's context bounds the whole wait, including temporary root-wallet unavailability. Without a caller deadline, one default 30-second deadline covers the entire call. Cancelling the wait does not cancel the shared receiving worker; the existing lifecycle must start that worker.

## Fees and Placement

`FREE_LOCAL` requires a valid fee proof and finite TTL, subject to signer / node quotas. It is available only at the current endpoint and does not enter P2P or network snapshots. An ordinary Seq + 1 write with CAS, a new AUTOPAY proof, and a new signature can upgrade it to paid storage; paid records cannot downgrade to FREE_LOCAL.

`AUTOPAY / PAID` availability and retention depend on FeeVerifier and contract state. Network-visible records participate in P2P. Grace after stopped payments is local retention behavior, without deletion history. Missing funds must not silently convert paid data to FREE_LOCAL. The SDK retains the original funding-paused outbox and prompt while continuing to receive subscription updates.

Omissions from a network snapshot do not remove local AUTOPAY grace data early. Retained data remains outside the network root and relay. A valid current record for the same key can still replace the cached copy; the existing retention cleanup deletes it when grace expires.

After payment resumes, an old expiry decision cannot delete the record. Successfully written or recreated current records publish their payment verification together. Concurrent maintenance updates only results matching current records and cannot remove a new record's valid network synchronization eligibility.

AccountBound mailboxes reside at the account's currently bound CoreNode. Message append / delete uses MessageManager; ordinary DKVS PUT cannot bypass the message protocol.

## Node P2P Synchronization

P2P transmits signed operations and current-set snapshots, without Wallet generations. Source eligibility derives from valid CoreNode / Bootstrap identity and configuration. A source is pinned only within one sync session, never persisted as collection ownership.

Current flow:

```text
Connect to an eligible source → extract directory hints from signed pages without retaining complete records
→ merge local directories → node-wide deduplicated queue
→ one source/session downloads directory snapshots at a time
→ Notify from eligible sources enters a node memory queue during synchronization
→ install all directories → replay cached Notify in arrival order
→ check the empty queue and set node READY under the same ingress lock
```

READY belongs to the entire node. Completion of one peer or directory cannot open notification processing early. Failure or disconnection retains unfinished directories. The Notify cache has count / byte limits and overflow requires resynchronization.

Ongoing Notify subscriptions for non-mining connections come from completed directory discovery/subscription declarations. One directory snapshot controls only its serving scope and does not change ongoing subscriptions; an initial standalone directory request does not establish an ongoing subscription either. During serving, the temporary scope can receive Notify. Completion, cancellation, or expiry retains only declared ongoing subscriptions. Mining connections and `/account` binding notifications follow their routing exemptions. Failed discovery-response validation rejects the page, retains unfinished work, and enters existing synchronization scheduling and source handoff.

The source captures one independent snapshot on the directory session's first request, then serves every page from it. Height advances or new writes cannot change that snapshot; the receiver replays cached Notify after installation. ViewHeight remains the capture-time record validation height, without a per-page comparison to current chain height. The receiver separately verifies payment eligibility at its current local height and confirms that validation still applies before installation, without changing the source snapshot boundary. Each connection retains at most one directory snapshot, capped at 64 MiB of records, released on completion, cancellation, disconnect, or two minutes of inactivity.

A repeated first-page request with the same active session and filters reuses the captured snapshot; a retry cannot change its subscription scope. The receiver ignores a response whose contents and signature exactly match the previously authenticated page in that session, without installing it twice or cancelling the next-page timeout. Different contents, invalid signatures, and other sessions retain existing validation rules. Connections process and send messages in order, so the receiver retains only the previous accepted response digest, without page history.

Complete installation validates roots, source signatures, scope, record permissions / fees, and the receiver's concurrent-update baseline. Each outstanding page is attempted twice against the same source, with a ten-second timeout per attempt. After both time out, existing connection selection prefers another eligible source, falling back to the original if none is available. Unfinished directories remain queued, and a DKVS timeout does not close the connection. Discovery supplies path hints without requiring one global root across its pages; it never directly installs or deletes KV. Complete directory validation remains unchanged. Notify or later anti-entropy can discover new directories. Discovery messages carry records, but the receiver only extracts directory paths and does not install records directly from discovery pages.

## Domain Integration Boundaries

Account management defines recovery packages, root protection, multi-device conflict handling, and wallet inventory. RGB11 defines encrypted snapshots / heads, business revisions, and asset-evidence validation. Domains access data through the SDK coordinator; DKVS eventual consistency does not imply automatic domain merging or asset validity.

Wallet-independent primitives, codecs, replicas, and outbox storage live in `sdk/wallet/dkvs`. Manager lifecycle, endpoint selection, and domain adapters live in the parent `wallet` package. The PWA calls domain-level WASM APIs and does not maintain its own DKVS transport.

See [RGB11 Assets and Wallet SDK](../rgb11/readme.md) for asset-validation and backup boundaries.
