# DKVS Technical Whitepaper

DKVS, the Distributed Key-Value Store, is the small-data storage layer embedded in SatoshiNet. It is designed for wallets, dApps, local Agents, D-Indexer workflows, and node coordination, with signed records, key-level permissions, and native node synchronization.

DKVS does not replace Bitcoin L1 and does not turn arbitrary off-chain data into consensus state. Its purpose is to let SatoshiNet nodes synchronize small verifiable records through native P2P messages, without libp2p, Kademlia, or the legacy L1 DKVS network layer.

## Goals

1. Provide one record model for user data, names, services, mailbox data, blob manifests, temporary data, and system data.
2. Require each record to carry pubkey, signature, TTL, expiry height, seq, fee proof, and optional tombstone flag.
3. Re-validate key shape, signature, TTL, permission, fee proof, hash, and selector rules on both local REST writes and remote P2P data.
4. Synchronize miners through SatoshiNet-native wire messages only.
5. First guarantee convergence among nodes that store a record; later phases may move from miner/core full storage to ordinary-node subscriptions and eventually DHT-style storage groups.

## Key Namespace

DKVS keys use path syntax:

```text
/<namespace>/<segments...>
```

Current namespaces:

| Namespace | Purpose | Write Permission |
| --- | --- | --- |
| `/personal/<account_id>/...` | Personal user data | `sha256(pubkey)==account_id` |
| `/name/<name_id>` | Name-level profile | Current L1 Ordinals DID/NS signing key or owner address |
| `/svc/<service>/...` | Service config and discovery | Current service signing key or owner address |
| `/mail/<mailbox>/msg/<msg_id>` | Offline messages | Any valid signer may deliver; quota is later work |
| `/mail/<mailbox>/share/<package>/<share>` | Guardian/share data | Mailbox owner only |
| `/blob/<object>/manifest` and `/blob/<object>/chunk/<n>` | Small blob chunks | Signer owns the record; SDK verifies manifest/chunks |
| `/tmp/...` | Temporary data | TTL-limited |
| `/sys/...` | System parameters, miner/pool metadata, and other system data | Configured system signer only |

Segments are restricted to `[a-z0-9._-]` and length limits. DKVS-safe names are used directly as `name_id`; unsafe canonical names use `hex(sha256(canonical_name))`.

## Records and Selection

Each key stores one active candidate, not version history.

If the same key already exists and the pubkey is unchanged, DKVS treats that pubkey as the established owner and avoids an unnecessary resolver call. If the local indexer marks the name as transferred, or if the new record uses a different pubkey, the next write must resolve current ownership again.

Selection rules:

1. A stale record whose current permission is invalid cannot block the new owner.
2. If the existing record is still valid, compare by `seq -> expiry_height -> hash/bytes`.
3. Tombstone is a delete record for the same key and still requires owner permission.
4. Get/List/Sync/Checkpoint return only unexpired active records whose current permission remains valid.

## DID / Name Resolver

DKVS core uses the `DIDResolver` interface for real DID/NS ownership. The recommended production path is the L1 indexer NS API:

```text
GET /ns/name/:name
GET /ns/address/:address
```

`/ns/name/:name` returns the current owner address. DKVS derives a p2tr address from the record pubkey and requires it to match the current owner. When a name moves with its UTXO, the L1 indexer can call the local SatoshiNet indexer method `NotifyDKVSNameTransfers(names)`, marking those names so the next write must resolve again.

Without a configured resolver, `/name` and `/svc` remain unwritable by default.

## Fee Proof and Storage Fees

Each DKVS record may carry a `fee_proof` proving that its storage cost is covered by a payment or authorization mechanism. Fee proof does not change transaction consensus semantics. It is part of DKVS write validation, and nodes re-validate it for local REST writes, incoming P2P `dkvsdata`, and startup sync imports.

The current fee proof is a JSON structure. Core fields include:

| Field | Meaning |
| --- | --- |
| `mode` | Payment mode. Current modes are `AUTOPAY`, `FREE_LOCAL`, `ONESHOT`, and `LEASE` |
| `pool_contract` | DKVS Pool or AUTOPAY contract address |
| `payer` | Payer address; in AUTOPAY it must match the p2tr address derived from the record pubkey |
| `payer_pubkey` | Optional payer pubkey; when present, it must derive to `payer` |
| `key_hash` | `sha256(key)` / DKVS key hash, binding the proof to the key |
| `record_hash` | `FeeAnchorHash(record)`, binding the proof to record content; it excludes record signature and the proof's own `record_hash` / `proof_signature` to avoid self-reference |
| `record_size` | Record size covered by the proof; AUTOPAY charges by full-size records |
| `expiry_height` | Expiry height covered by the proof |
| `namespace` | Key namespace; it must match the actual key namespace |
| `proof_signature` | Optional payer signature over canonical fee proof fields; mainnet defaults require it |

Current modes:

| Mode | Current Use | Mainnet Boundary |
| --- | --- | --- |
| `AUTOPAY` | Recommended current mode. The writer provides an `autopay.tc` template contract address, and the node reads contract state to calculate this payer's record capacity. | Not accepted by default on mainnet until production parameters are finalized. |
| `FREE_LOCAL` | Local testing, development, or explicit whitelist policy. | Not accepted by default on mainnet; expired records may be physically deleted. |
| `ONESHOT` | Reserved one-time payment format with `payment_txid` and `paid_amount`. | Current code only performs JSON field validation; real DKVS Pool payment verification is later work. |
| `LEASE` | Reserved lease/plan payment format with `lease_contract` and `plan_id`. | Current code only performs JSON field validation; real lease contract verification is later work. |

### AUTOPAY Verification

AUTOPAY is the currently practical fee proof path. A writer submits the `autopay.tc` template contract address with the record. The node reads contract state and verifies:

1. The contract is the `autopay.tc` template.
2. The contract is active and not closed.
3. The deployer is the record writer payer.
4. If `payer_pubkey` is present, it must derive to the `payer` p2tr address.
5. The record pubkey must equal `payer_pubkey`, and its derived p2tr address must equal `payer`.
6. Recipient, fee asset, and end height match DKVS policy.
7. The per-block payment covers the active full-size record count currently assigned to that AUTOPAY contract.

Capacity is calculated as:

```text
max_records = floor(amount_per_block / full_record_fee_per_block)
```

For fixed schedules, `amount_per_block = base_amount`. For linear schedules, `amount_per_block = base_amount + step_amount * max(0, current_block - active_height - 1)`.

Testnet may hard-code a default DKVS AUTOPAY policy with a fixed deployer, recipient, fee asset, base amount, and full-record fee to derive the default contract address. These defaults do not by themselves authorize writes; the node must still read a live active contract state. Mainnet does not allow free writes and does not accept records from a default contract until production contract parameters, recipient, and settlement rules are finalized.

### DKVS Pool and Mining Revenue

DKVS storage fees are intended to be merged into mining rewards. The target design is that the DKVS Pool contract automatically emits one result tx in each block, and the fee of that result tx represents all record-storage fees owed to the miner for that block. With this model, DKVS core only verifies that a record is covered by a valid fee proof; it does not calculate or distribute rewards in the P2P synchronization layer.

The DKVS Pool contract is not implemented yet. The current AUTOPAY verifier only checks contract state and record capacity coverage; it does not mean final revenue distribution is live. Mainnet fee policy should remain disabled until the Pool contract, result tx format, and settlement rules are finalized.

## Retention and Expiration Pruning

DKVS separates the active view from physical retention:

- only active records are returned by `GET`, prefix list, sync, checkpoint, and snapshot;
- after TTL or `expiry_height` expires, the record leaves the active view;
- records that have previously paid storage fees are physically retained for now, so they can later support renewal, audit, or recovery flows;
- free records, including local `FREE_LOCAL` records or records accepted without a fee proof by local policy, may be physically deleted after expiration.

Nodes must provide a DKVS-side timer that periodically scans and deletes expired free records. The current implementation also triggers the same cleanup path periodically from block processing. This does not change the wire protocol and does not physically prune paid records.

## P2P Synchronization

DKVS uses six native SatoshiNet wire commands:

| Command | Purpose |
| --- | --- |
| `dkvsnotify` | Notify key/hash updates |
| `dkvsinv` | Broadcast record inventory |
| `dkvsget` | Fetch by key/hash |
| `dkvsdata` | Return record data |
| `dkvssyncreq` | Paged startup sync request |
| `dkvssyncres` | Paged sync response |

Miners perform paged sync when peers connect. After notify, receivers fetch missing records through get/data. The receiver always re-validates the record.

## Checkpoint

A DKVS checkpoint is a Merkle root over the local active record view. It helps nodes compare views and export snapshots. It is not a consensus root.

It answers one narrow question: whether two nodes currently see the same DKVS active record set. A sync response can carry a checkpoint root; after applying the page, the receiver recomputes its local root, and a mismatch means it should continue syncing, retry, or alert. A checkpoint does not hold private keys, does not require a signature, does not prove that all historical data exists forever, and does not automatically create chain transactions. The current design does not publish checkpoints as DKVS records; the SatoshiNet indexer itself does not hold checkpoint signing private keys.

Checkpoint signing and chain anchoring have been removed from the current design. Long term, not every node will store all DKVS data: core nodes may store all records, ordinary nodes may store subscribed records, and very large deployments may store records by DHT-style groups. A mandatory global anchor would be expensive and may provide misleading assurance. The more useful guarantee is that every node responsible for a given record eventually receives its updates.

## Current Boundary

The current stage does not include ordinary-node subscription markets, mailbox quota, large blob SDK, DKVS Pool contract/result-tx revenue settlement, chain checkpoint anchoring, or DHT storage groups. These remain later protocol stages.
