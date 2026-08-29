# DKVS: SatoshiNet Distributed Key-Value Storage

DKVS, the Distributed Key-Value Store, is SatoshiNet's owner-controlled small-data storage and synchronization layer. It provides a common signed-record model for wallets, account recovery, encrypted RGB11 backup, mailbox delivery, service discovery, and application configuration.

DKVS is not a general-purpose multi-primary database and is not Bitcoin or SatoshiNet consensus state. It addresses a narrower problem: **how a service node validates writers, accepts updates atomically, and lets relayable data converge through the node protocol, while Wallet maintains a confirmed local replica of the current service endpoint.**

## Core Principles

1. A key expresses stable business identity, not an implementation or schema version.
2. `Seq` orders updates to one key, and `ETag = RecordHash(record)` controls per-key concurrency.
3. Wallet writes use only `ExpectAbsent` or `ExpectedETag`; they do not use path roots, path generations, or endpoint epochs.
4. Batch CAS commits atomically at one service node. A failed mutation never leaves a partial batch.
5. `FREE_LOCAL` is a fee and endpoint-local retention mode, not a second application protocol.
6. A Wallet keeps one aggregated prefix long-poll per service endpoint and reads subscribed data from its local confirmed replica.
7. `PathMeta`, `StateRoot`, network delete floors, and path snapshots belong only to canonical node-to-node synchronization.
8. Wallet SDK `dkvsManager` is the single owner of transport, prefix registry, replica, cursor, outbox, and background synchronization.
9. DKVS does not understand or automatically merge account-management, RGB11, or other domain objects. Each domain defines its own conflict policy.
10. The first release switches directly to the final protocol and local schema. It keeps no dual reads, dual writes, legacy decoder, or versioned-key fallback.

## Stable Keys and Namespaces

DKVS keys use path syntax:

```text
/<namespace>/<segments...>
```

Primary namespaces:

| Namespace | Purpose | Write authority |
| --- | --- | --- |
| `/account/...` | Public account mappings and profiles | Corresponding account identity |
| `/personal/<account_id>/<application>/...` | Wallet, account-management, RGB11, and other private domain data | Account owner |
| `/blob/<account_id>/<blob_name>` | Encrypted snapshots or bounded larger objects | Account owner |
| `/mail/<receiver>/msg/<sender>/<msg_id>` | Offline messages | Sender creates; receiver may delete |
| `/mail/<receiver>/share/...` | Guardian/share data | Receiver or protocol-selected writer |
| `/name/<name>` | Name profile | Current DID/NS authority |
| `/svc/<service>/...` | Service configuration and discovery | Current service authority |
| `/tmp/...` | Temporary application data | Bounded TTL and policy-defined local-only storage |
| `/sys/...` | System parameters | Configured system signer |

Correct keys directly name business objects:

```text
/personal/<account_id>/wallet/catalog
/personal/<account_id>/wallet/settings
/mail/<account_id>/msg/<sender_id>/<msg_id>
/blob/<account_id>/<blob_name>
```

A data-structure upgrade for the same object uses the same key and a higher `Seq`. A version may appear in a key only when an external protocol version is itself part of business identity and multiple generations must remain selectable at the same time. Every exception requires explicit approval, a minimal allowlist entry, and dedicated tests.

## Signed Records and ETags

DKVS keeps one signed-record format:

```text
Version
Key
Value
Seq
IssueHeight
TTL
FeeProof
Flags
PubKey / account identity
Signature
```

`ExpiryHeight` is derived from record-height and TTL semantics. The ETag is:

```text
ETag = RecordHash(record)
```

For a deleted key, its ETag is the hash of the latest signed tombstone or the service node's effective delete floor.

Primary size boundaries are:

- ordinary values are limited to 16 KiB;
- `/blob` values are limited to 1 MiB;
- one batch contains at most 64 mutations and 8 MiB of encoded records;
- a key is limited to 256 bytes and one segment to 64 bytes;
- subscription snapshots have separate record-count and total-byte limits.

DKVS is not a generic file store. Large files should use a dedicated distribution system. DKVS should primarily store small objects, encrypted snapshots, manifests, metadata, or stable references.

## Four Consistency Boundaries

| Problem | Mechanism | Layer |
| --- | --- | --- |
| Per-key concurrency | `Seq` + `ETag` | Wallet and service node |
| Wallet delta position | Endpoint-scoped opaque cursor | Wallet subscription |
| FREE_LOCAL ownership | Stable `EndpointID` | Current service node |
| Canonical node convergence | `PathMeta` + `StateRoot` | Internal SatoshiNet P2P |

Each mechanism solves one problem. Wallet requests never carry `ExpectedPathRoot`, `ExpectedPathGeneration`, a `ViewHeight` CAS condition, or endpoint generation.

## Fees and Storage Modes

The service node derives `FREE_LOCAL`, `AUTOPAY`, or another paid mode from `FeeProof`. APIs may return the derived `storage_mode`, but it is not a second persistent source of truth.

### AUTOPAY and Other Paid Modes

`AUTOPAY` is the primary relayable storage mode. The node validates the contract, service name, fee asset, recipient, signer-linked delegation, active-delegate state, per-block amount, balance, and capacity. A valid canonical record may enter node-to-node replication.

### FREE_LOCAL

`FREE_LOCAL` has strict semantics:

- it belongs only to the storage instance identified by the current `EndpointID`;
- it is a single-endpoint, single-writer mode;
- it carries a valid FREE_LOCAL fee proof;
- TTL, per-signer quota, total endpoint quota, and blob limits are enforced by endpoint policy;
- it never enters P2P relay, canonical snapshots, checkpoints, or anti-entropy;
- it does not support lossless endpoint switching;
- UI must not describe it as network-wide backup or cross-node persistence.

Allowed transitions are:

```text
absent            -> FREE_LOCAL
absent            -> AUTOPAY/PAID
FREE_LOCAL        -> FREE_LOCAL
FREE_LOCAL        -> AUTOPAY/PAID
AUTOPAY/PAID      -> AUTOPAY/PAID
```

The following transition is forbidden:

```text
AUTOPAY/PAID -> FREE_LOCAL
```

`FREE_LOCAL -> AUTOPAY/PAID` does not use a promote API. Wallet reads the current `Seq/ETag`, creates a `Seq+1` record with a new fee proof and signature, and sends an ordinary `Put(ExpectedETag)`. The service node switches that key to canonical storage in one database batch and relays it only after commit.

## CRUD and Per-Key CAS

The final Wallet application API, relative to the existing DKVS base path, is:

```text
GET  /config
GET  /record?key=...
GET  /key-state?key=...
POST /records/batch-cas
POST /subscriptions/snapshot
POST /subscriptions/watch
```

### Read

For a subscribed prefix:

```text
Get(key)
-> read local confirmed replica
-> evaluate expiry at the last trusted ViewHeight
-> return value and freshness
```

An unsubscribed key is read directly from the service node while online and is unavailable while offline. An ordinary `Get` never creates a permanent subscription.

### Put and Update

A write carries only:

```text
request_id
exact signed record
ExpectAbsent or ExpectedETag
EndpointID when the batch contains FREE_LOCAL
```

The service node validates the current effective key state, precondition, continuous `Seq`, signature, namespace permission, fee mode, TTL, and quota, then atomically commits the record and change commit.

Wallet may apply a successful write response immediately and idempotently, but it must not jump the subscription cursor to the cursor returned by that write. Background synchronization continues from the previous subscription cursor, consumes the complete commit order, and deduplicates the same ETag or RequestID.

### Delete and Recreate

Delete is an ordinary signed tombstone Put:

```text
Flags = Tombstone
Value = empty
Seq = current floor + 1
ExpectedETag = current ETag
```

A FREE_LOCAL delete creates only an endpoint-local floor; a canonical delete enters the node-to-node delete floor. Before recreating a deleted key whose history is absent locally, Wallet calls `/key-state` for `active / deleted / never_seen`, the latest `Seq`, and the current ETag.

### Multi-Key Batch

A batch may contain several keys under one logical owner, with an independent ETag precondition for each mutation. All preconditions, records, delete states, internal PathMeta changes, and one change commit are written in one service-node database batch. A failure applies nothing. An exact signed request is replayable as-is; partial prior application indicates a server atomicity failure.

## Durable Change Log and Opaque Cursor

Every effective-view change at the current endpoint must append a change commit in the same transaction, including:

- FREE_LOCAL or paid Put;
- explicit Delete;
- TTL or retention expiry;
- ordinary-Put transition from FREE_LOCAL to AUTOPAY;
- an incoming P2P canonical update;
- a P2P mirror or path snapshot that changes the endpoint effective view.

Failed transactions and real no-ops write no change and wake no waiter. One multi-key batch maps to one atomic commit.

The external cursor is an opaque token bound to `EndpointID`, revision, the previous token, and commit history. A client must not parse or construct it, and it must not retain only a bare revision. `RESET_REQUIRED` or `ENDPOINT_MISMATCH` is returned when:

- the cursor belongs to another endpoint;
- it is unknown, compacted, ahead, or tampered with;
- the revision matches but the token does not;
- restoring an older database backup creates a different change history.

A normal restart restores EndpointID, record state, change log, and head token together, so existing cursors remain valid. Cursor reset is a normal recovery path: Wallet fetches a new snapshot. No endpoint epoch is needed.

## Explicit Prefix Subscription

Wallet SDK explicitly registers, removes, lists, and reports the status of prefixes. The registry is durable and resumes after Wallet restart. Accessing one key never implicitly and permanently subscribes its path.

On initial subscription or reset, the service node reads the following under one database snapshot, read transaction, or consistency lock:

```text
endpoint effective records
+ current change-log head cursor
```

Changes before the snapshot are in its records, while changes after the snapshot appear after its cursor. There is no snapshot-to-watch loss window. Wallet validates record signature, key, identity, Seq, and expiry. It does not save or verify PathMeta, StateRoot, PathGeneration, EndpointPathState, or a local overlay.

One terminal keeps one aggregated long-poll to one service endpoint. The request carries the complete prefix set and one cursor. Default or recommended boundaries are:

```text
MaxPrefixesPerTerminal = 16
MaxPrefixLength        = 256
MaxChangesPerResponse  = 256
LongPollTimeout        = 60 seconds
ReconnectJitter        = 0-1 second
```

The service indexes `topic -> waiter references` and wakes only topics affected by a mutation. It must not scan all terminals, create one long-poll per prefix, or create a periodic ticker per request. After registering a waiter, it rechecks backlog to close the query-to-wait race.

Responses preserve atomic commit groups. Wallet applies all keys in one commit through one local database batch, writes the commit cursor last, and notifies domain observers only after success.

## Wallet Local Replica and Outbox

`dkvsManager` is the only DKVS coordination layer. Domain modules access logical keys or domain APIs; they do not own a DKVS client or implement restore, conflict, remote synchronization, or auto-backup workers. PWA JavaScript does not know the DKVS transport.

The local replica stores only:

```text
dkvs:subscription-state
dkvs:subscription-prefix:<prefix>
dkvs:subscription-record:<key>
dkvs:subscription-key-state:<key>
dkvs:outbox:<request_id>
```

Subscription status is:

```text
SYNCING
READY
OFFLINE_READY
RESET_REQUIRED
ERROR
```

The replica does not store PathMeta, PathRoot, PathGeneration, separate network/local delete floors, EndpointGeneration, HasLocalOnly, or path-session state. The first release also keeps no old-schema fallback or automatic migration. Development cache, replica, subscription-state, and outbox data may be cleared and rebuilt.

The durable outbox uses unique `RequestID` keys and stores exact signed mutations, per-key preconditions, state, attempt count, and errors. A request containing FREE_LOCAL is pinned to the current EndpointID. After a network timeout, the exact entry is replayed; it is not modified, re-signed, or given a new precondition. After conflict, the domain recalculates from fresh key state and creates a new RequestID and signed mutation.

The server change log, Wallet replica, and outbox are durable data and therefore use compact deterministic binary encoding with strict decode validation. JSON is reserved for HTTP APIs, logs, and human diagnostics; API JSON must not become a persistence format unless an explicit compatibility requirement is documented.

## Offline Read and Endpoint Switching

Local subscription state retains the last trusted `ViewHeight`:

- a record is hidden when that trusted height has reached its expiry;
- when chain movement during offline time is unknown, freshness is `offline_last_known`;
- wall-clock time never fabricates block height;
- offline readability means the last confirmed state is available, not that it is currently network-latest.

Before switching service endpoints, Wallet scans active FREE_LOCAL records and pending FREE_LOCAL outbox entries. Required data is first changed to canonical storage with an ordinary AUTOPAY Put, then Wallet waits for write success, empty outbox, and observation of the same ETag through the current subscription.

On the new endpoint:

```text
obtain the new EndpointID
-> old cursor becomes invalid naturally
-> fetch a new snapshot for every explicit prefix
-> atomically install the new replica and cursor
-> complete the switch
```

Until the new snapshot is installed, the old replica may be used only as `offline_last_known`; it cannot be marked READY for the new endpoint.

## Node-to-Node P2P Boundary

Nodes continue to maintain internally:

```text
canonical PathMeta / StateRoot
network tombstone / delete floor
trusted PathSnapshot
Notify / Inv / Get / Data
anti-entropy
ordinary-node subscription ranges
complete Miner canonical data
```

The Wallet change log is an endpoint application-view delta and does not replace the P2P canonical root. Small P2P changes may emit ordinary change commits; large replacements may emit `RESET_PREFIX`. A canonical snapshot or mirror must not delete endpoint-local FREE_LOCAL keyspace, and the complete PathMeta/snapshot structure is never sent to Wallet.

## Domain Boundaries

### Account Management

Account management uses stable `/personal/<account_id>/account/...` keys. A recovery package and related objects may be committed through one multi-key batch. Multi-device field merging, wallet inventory, root-wallet protection, and recovery policy belong to the account-management domain, not to a general DKVS multi-primary guarantee.

### RGB11

RGB11 uses separate `/personal/<account_id>/rgb11/...` and `/blob/<account_id>/...` keys:

- encrypted snapshot and wallet head use one key-ETag batch CAS;
- the head binds a monotonic domain revision, snapshot state hash, and operation ID;
- FREE_LOCAL supports restore only through the same endpoint;
- cross-node persistence is enabled by an ordinary AUTOPAY Put;
- a stale writer receives a head or DKVS write conflict and cannot overwrite newer state;
- RGB asset validity remains a client-validation and Bitcoin-evidence decision.

See [RGB11 Assets and Wallet SDK](../rgb11/readme.md) for the asset-specific model.

## Capacity Target and Acceptance

The first-release target is at least 10,000 realtime terminals per service node, primarily for private prefixes. At an average of four to eight prefixes per terminal, there are still 10,000 active long-polls, not one connection per prefix. Public hot prefixes and unbounded large blobs are outside this phase's capacity SLA and require a later hot-distribution design.

Ten-thousand-terminal capacity is an acceptance target, not something inferred from goroutine counts. Before release, testing must include at least:

- staged runs at 100, 1,000, 5,000, and 10,000 terminals;
- 10,000 terminals with five prefixes each and 60-second long-polls for 30–60 minutes;
- idle request rate near `10,000 / 60 ≈ 167 RPS`, without linear growth in prefix count;
- P99 notification latency below two seconds when 1,000 private prefixes change in a short interval;
- no waiter, FD, goroutine, heap, or topic-reference leak after timeout, cancel, restart, and compaction;
- race-detector, fault-injection, and target-hardware checks for CPU, RSS, database IOPS, and network headroom.

## Stable Error Codes

Wallet branches on typed errors or stable codes, never human-readable text. Primary application errors include:

```text
DKVS_WRITE_CONFLICT
DKVS_ENDPOINT_MISMATCH
DKVS_RESET_REQUIRED
DKVS_PERMISSION_DENIED
DKVS_INVALID_SEQUENCE
DKVS_LOCAL_ONLY_ENDPOINT_MISMATCH
DKVS_STORAGE_MODE_DOWNGRADE
DKVS_QUOTA_EXCEEDED
DKVS_RECORD_NOT_FOUND
```

Internal node synchronization may still use stale-generation or path-divergence errors, but those are not Wallet CRUD preconditions.

## Explicit Boundaries

DKVS first release does not provide:

- arbitrary multi-primary CRDT semantics;
- cross-account transactions;
- cross-node linearizable commits;
- quorum, BFT, or an on-chain commit certificate;
- FREE_LOCAL recovery across endpoints;
- downgrade from PAID/AUTOPAY to FREE_LOCAL;
- a promote API or endpoint epoch;
- general large-file storage;
- a 10,000-terminal fan-out SLA for public hot prefixes;
- automatic understanding and merging of arbitrary business objects;
- compatibility with the old Wallet PathMeta protocol, old local schema, or versioned keys.

Applications should treat DKVS as a verifiable, owner-controlled, eventually consistent small-data layer, not as a relational database or a global consensus database.
