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

DKVS records are kept compact. A record has one payload field, `Value`; there is no separate `Data` field. The maximum encoded record size is 16KB. `FeeProof`, pubkey, signature, and metadata all count toward that limit, so applications should reserve most of the space for `Value`.

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

The current fee proof is a compact binary structure stored in the record's `FeeProof` bytes field. The record signature covers `FeeProof`, so the proof no longer carries its own signature, record hash, key hash, record size, expiry height, or namespace fields. Those values are derived from the record or supplied by the verifier context.

Current proof modes and encoded content:

| Mode | Encoded Content | Current Use |
| --- | --- |
| `AUTOPAY` | `pool_contract` | Recommended current mode. The node reads the `autopay.tc` template contract state to calculate the record signer's capacity. |
| `FREE_LOCAL` | No extra fields | Local testing, development, or explicit whitelist policy. |
| `ONESHOT` | `pool_contract`, `payer`, `payment_txid`, `paid_amount` | Reserved one-time payment proof format. |
| `LEASE` | `pool_contract`, `lease_contract`, `plan_id` | Reserved lease/plan proof format. |

Mainnet does not accept `FREE_LOCAL` by default. `ONESHOT` and `LEASE` currently provide compact encoding and basic field validation only; real one-shot payment and lease-payment verification are later work.

### AUTOPAY Verification

AUTOPAY is the currently practical fee proof path. A writer submits the `autopay.tc` template contract address with the record. The node reads contract state and verifies:

1. The contract is the `autopay.tc` template.
2. The contract is active and not closed.
3. The node derives a p2tr address from `record.PubKey` and treats that address as the delegate payment address for the current record.
4. The contract service name, recipient, fee asset, and minimum payment amount match DKVS policy.
5. That delegate address has an active delegate configuration in contract state.
6. That delegate address's per-block amount covers the full-size active records it has written.
7. That delegate address's funding balance is sufficient for the next block.

Capacity is calculated as:

```text
max_records = floor(delegate_amount_per_block / full_record_fee_per_block)
```

`delegate_amount_per_block` comes from the independent configuration for that delegate address in `autopay.tc` runtime state. If the delegate has not configured an amount explicitly, runtime uses the minimum amount set at deployment.

Testnet may hard-code a default DKVS AUTOPAY policy with a fixed service name, recipient, fee asset, minimum amount, and full-record fee to locate the default contract and calculate capacity. These defaults do not by themselves authorize writes; the node must still read a live active contract state, and the delegate address derived from the record signer must have a valid payment configuration and balance. Mainnet does not allow free writes and does not accept records from a default contract until production contract parameters, recipient, and settlement rules are finalized.

### AUTOPAY Payment Pool and Mining Revenue

DKVS storage fees are intended to flow into mining rewards or into a configured service recipient. In the first stage, `autopay.tc` acts as the DKVS payment pool: multiple delegate addresses fund payment assets into one contract, and each block the contract aggregates all active delegates' payable amounts into one result tx.

If the `autopay.tc` recipient is empty, the aggregated payment is treated as miner fee and enters the block reward. If the recipient is not empty, the aggregated payment output goes to the configured service recipient. DKVS core only verifies that a record is covered by a valid fee proof; it does not calculate or distribute rewards in the P2P synchronization layer.

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

The current stage does not include ordinary-node subscription markets, mailbox quota, large blob SDK, complete one-shot or lease-payment settlement, chain checkpoint anchoring, or DHT storage groups. These remain later protocol stages.
