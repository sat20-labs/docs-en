# DKVS: SatoshiNet Distributed Key-Value Storage

DKVS, the Distributed Key-Value Store, is SatoshiNet's owner-controlled small-data storage and synchronization layer. It gives wallets, account recovery, RGB11 state backup, mailbox delivery, service discovery, and application configuration a common signed-record model.

DKVS is not a general-purpose multi-primary database and is not Bitcoin or SatoshiNet consensus state. It addresses a narrower problem: **how nodes responsible for a record validate its writer, accept updates atomically, and eventually converge on the same effective state.**

## Core principles

1. An ordinary logical path has one owner or authority.
2. `Seq` orders revisions of one key; `PathGeneration` orders mutations of the logical path.
3. Writes use CAS or batch-CAS. A failed precondition never produces a partial commit.
4. The signature covers the key, value, fee proof, time, sequence, and path generation.
5. Relayable data propagates through SatoshiNet-native P2P messages; `FREE_LOCAL` data belongs only to the receiving endpoint.
6. Wallet SDK domain modules access DKVS only through `dkvsManager`, which owns transport, replica, synchronization, generation, and outbox state.
7. DKVS does not understand or merge account-management, RGB11, or other domain objects. A domain must define its own conflict policy.

## Keys and logical paths

DKVS keys use path syntax:

```text
/<namespace>/<segments...>
```

Primary namespaces:

| Namespace | Purpose | Logical path / permission |
| --- | --- | --- |
| `/personal/<account_id>/<module>/...` | Personal data, account management, RGB11 heads | Owner-exclusive path per module; only the account owner may write |
| `/blob/<account_id>/<blob_key>` | Encrypted snapshots or larger objects | Each complete blob key is an owner-exclusive path |
| `/mail/<receiver>/msg/<sender>/<msg_id>` | Offline messages | Shared append by sender subpath; receiver may delete |
| `/mail/<receiver>/share/...` | Guardian/share data | Receiver owner-exclusive |
| `/name/<name>` | Name profile | Current DID/NS authority |
| `/svc/<service>/...` | Service configuration and discovery | Current service authority |
| `/tmp/...` | Short-lived relay and ACK data | Local-only with bounded TTL |
| `/sys/...` | System parameters | Configured system signer |

Modules under `/personal/<account_id>` use separate logical paths. Account management and RGB11 therefore do not share one generation counter or one write lock.

### Path modes

| Mode | Semantics |
| --- | --- |
| `owner_exclusive` | The owner is derived from an account identifier; normally one active writer |
| `authority_exclusive` | The writer is selected by DID, service, or system authority |
| `shared_append` | Multiple writers create independent unique keys and do not share one mutable value |
| `local_only` | Stored only by the current endpoint and excluded from network PathMeta |

## DKVSRecord v1

A record contains:

```text
Version
Key
Value
PubKey
Signature
Seq
PathGeneration
IssueTime
TTL
ExpiryHeight
FeeProof
Flags
```

### Size limits

- An ordinary value is limited to 16 KiB.
- A `/blob` value is limited to 1 MiB.
- A blob is one complete record. The obsolete manifest/chunk protocol is not used.
- One batch may contain at most 64 mutations and at most 8 MiB of encoded records.
- A key is limited to 256 bytes and a segment to 64 bytes.

DKVS is not a generic file store. Large files should use a dedicated distribution system; DKVS should hold small objects, encrypted snapshots, or references.

### Seq and PathGeneration

A normal key update must satisfy:

```text
new.Seq = current.Seq + 1
```

Each effective mutation of a logical path uses:

```text
new.PathGeneration = current_path_generation + 1
```

Within one batch, records are assigned consecutive `PathGeneration` values in canonical key order. Remote nodes read the owner-signed generation from the record; they do not derive it from local arrival count.

### IssueTime and deterministic selection

Wallet SDK uses the endpoint's `server_time_ms` to produce monotonic issue time:

```text
IssueTime = max(server_time_ms, previous_issue_time + 1)
```

If abnormal concurrency produces multiple candidates for one key, the selector is:

1. greater `Seq`;
2. retention renewal when the business content is identical;
3. greater `IssueTime`;
4. byte order of `RecordHash`.

This makes convergence deterministic. It does not guarantee that every business edit survives unsupported concurrent writers.

## PathMeta and synchronization

Network-comparable PathMeta contains:

```text
Path
Generation
StateRoot
ActiveRecords
ActiveTotalSize
MinExpiryHeight
ViewHeight
```

`StateRoot` is a deterministic digest of effective records and delete floors in the path. It is a synchronization signal, not an on-chain commitment and not a Merkle membership proof.

Comparison rules:

1. the endpoint with lower generation needs synchronization;
2. equal generation and root means the path is converged;
3. equal generation with a different root requires full reconciliation;
4. an endpoint below the client's confirmed generation is stale and cannot accept another write for that path.

A full path snapshot includes PathMeta, effective records, delete floors, and `server_time_ms`. The receiver validates the entire snapshot and atomically replaces its confirmed replica.

## CAS and batch-CAS

A single-key CAS binds at least:

```text
signed_record
expected_path_generation
expected_record_hash or expect_absent
```

Batch-CAS provides local atomic submission for related keys owned by the same authority, including:

- the account recovery envelope, shares, questions, and manifest;
- an RGB11 encrypted snapshot and wallet head;
- application records that must change together.

Batch-CAS guarantees at the receiving RPC node:

- all validations succeed before one database commit;
- any failed mutation returns `applied=0`;
- multiple paths are locked in canonical order;
- an exact record or batch retry is idempotent;
- a partially pre-existing batch returns conflict instead of filling the missing subset.

It is not a cross-node linearizable transaction and does not support cross-owner transactions.

## Fees and retention

### AUTOPAY

`AUTOPAY` is the primary relayable storage mode. A node reads `autopay.tc` contract state and verifies:

- template, service name, fee asset, and recipient;
- the delegate address derived from the signer;
- active delegate status;
- per-block amount and balance;
- whether active record usage exceeds capacity.

Capacity is calculated using the full-record rate:

```text
max_records = floor(amount_per_block / full_record_fee_per_block)
```

### FREE_LOCAL

`FREE_LOCAL` supports development, temporary caching, and explicit endpoint-local backup:

- the record carries a valid FREE_LOCAL fee proof;
- it is stored only by the current endpoint;
- it is never P2P-relayed;
- it is excluded from network PathMeta, checkpoints, and path snapshots;
- TTL, record count, bytes, and blob-key count are bounded by endpoint policy;
- another device can recover it only through the same endpoint;
- after endpoint switching, UI must not describe it as a network backup.

Whether a production node enables FREE_LOCAL is a node policy decision.

### Other proof modes

Compact encodings exist for `ONESHOT` and `LEASE`, but their complete settlement verification remains later work.

## P2P and the endpoint-local overlay

Relayable records propagate through native SatoshiNet DKVS messages. Every receiving node re-validates:

- key and namespace;
- owner or authority;
- signature and fee proof;
- sequence, PathGeneration, and delete floor;
- size, TTL, expiry, and quota.

A generation gap cannot be guessed or locally filled. The node marks the path stale and performs full path synchronization.

FREE_LOCAL records do not appear in the network snapshot. After validating the network path snapshot, Wallet SDK separately reads local-only records from the same endpoint and merges them into an endpoint-scoped overlay. That overlay never affects the network `StateRoot`.

## Wallet SDK dkvsManager

Domain modules do not own a DKVS transport client. `dkvsManager` coordinates:

- endpoint clients and endpoint identity;
- per-path locks and readiness;
- confirmed replicas and local-only overlays;
- sequence, PathGeneration, and monotonic IssueTime;
- CAS and batch-CAS;
- exact signed-batch outbox entries;
- refresh, watch, and change notification;
- typed error mapping.

Write flow:

```text
wait for path readiness
→ acquire per-path lock
→ read confirmed replica and PathMeta
→ allocate Seq and PathGeneration
→ sign the exact record or batch
→ persist exact outbox bytes
→ submit CAS or batch-CAS
→ apply the write response to replica, PathMeta, and outbox
→ notify the domain module
```

Retries reuse the exact signed bytes. They must not regenerate sequence, generation, time, or signature.

Stable error codes include:

```text
DKVS_WRITE_CONFLICT
DKVS_STALE_GENERATION
DKVS_STALE_ENDPOINT
DKVS_PERMISSION_DENIED
DKVS_INVALID_SEQUENCE
DKVS_PATH_DIVERGED
DKVS_LOCAL_ONLY_ENDPOINT_MISMATCH
DKVS_QUOTA_EXCEEDED
DKVS_RECORD_NOT_FOUND
```

Applications should branch on typed errors or stable codes, never human-readable error strings.

## Account management

Account management uses `/personal/<account_id>/account/...`:

- a recovery package is published as one atomic four-record batch;
- managed wallet state uses an encrypted envelope and monotonic revision;
- explicit synchronization refreshes the remote path first;
- write conflict, stale generation, path divergence, and invalid sequence use bounded retries;
- wallet names, account metadata, and newly enabled accounts are replayed as field-level domain mutations;
- wallet deletion is an inventory mutation and collapses earlier metadata mutations for that wallet;
- the root wallet cannot be deleted;
- a wrong secret or wrong root mnemonic cannot restore the state.

This field-level merge is account-management logic, not a general DKVS multi-primary guarantee.

## RGB11

RGB11 uses separate `/personal/<account_id>/rgb11/...` and `/blob/<account_id>/...` paths:

- encrypted snapshot and wallet head are committed in one batch-CAS;
- the head is a monotonic revision binding the snapshot state hash and operation ID;
- endpoint-local FREE_LOCAL backup can be restored by another device using the same endpoint;
- active AUTOPAY upgrades storage to a relayable backup;
- if AUTOPAY lookup fails but FREE_LOCAL policy is available, the wallet falls back to temporary backup;
- a stale writer returns head conflict and cannot overwrite a newer remote state;
- RGB asset validity remains a client-validation and Bitcoin-evidence decision. DKVS stores encrypted state and transport data only.

See [RGB11 Assets and Wallet SDK](../rgb11/readme.md) for the asset-specific model.

## E2E acceptance coverage

Wallet SDK E2E tests start local bootstrap, core, and miner nodes and cover:

- AUTOPAY name-owner rotation, mailbox append/tombstone, and three-node convergence;
- same-endpoint FREE_LOCAL recovery and cross-endpoint isolation;
- atomic account recovery publication;
- account-management activation, recovery, boundaries, and two-device field-level convergence;
- fixed-address RGB11 invoices, encrypted backup, same-endpoint restore, and stale-writer rejection;
- CAS, PathGeneration, PathMeta, typed errors, and P2P convergence.

Tests that connect to an existing public testnet, spend shared test assets, or mutate public network state use separate build tags and are excluded from the default suite.

## Explicit boundaries

DKVS v1 does not provide:

- arbitrary multi-primary CRDT semantics;
- cross-account transactions;
- cross-node linearizable commits;
- quorum, BFT, or an on-chain commit certificate;
- FREE_LOCAL recovery across endpoints;
- general large-file storage;
- complete ONESHOT or LEASE settlement;
- automatic merging of arbitrary business objects.

Applications should treat DKVS as a verifiable, owner-controlled, eventually consistent small-data layer, not as a relational database or a global consensus database.
