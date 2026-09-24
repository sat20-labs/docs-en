# RGB11 Assets and Wallet SDK

RGB11 is a client-validated asset protocol built on Bitcoin L1. SAT20 Wallet SDK places RGB11 contracts, asset state, UTXO proofs, transfer workflows, and recovery under a dedicated `rgb11Manager`, then exposes domain-level methods to PWAs, desktop wallets, and other applications.

The validity of an RGB11 asset is ultimately determined by its contract, consignment, single-use seals, Bitcoin transaction evidence, and client-side validation results. An Indexer may supply transaction and UTXO evidence, but it cannot replace RGB11 validation or manufacture a balance from index data.

> The current implementation completes the Bitcoin L1 wallet loop first. The STP path for bringing RGB11 assets into SatoshiNet is still under development. Until full STP support is available, the SDK treats STP-dependent operations as unavailable.

## 1. Protocol Version and Source Baseline

### 1.1 Adopted RGB version

SAT20 explicitly fixes the `rgb11` protocol space to the **RGB 0.11.1 family**. The consensus, operations, invoicing, schema, and PSBT/API baseline used by the current Go implementation is frozen at the **`0.11.1` stable release**.

The version relationship is:

- SAT20 protocol name: `rgb11`;
- protocol target: RGB `0.11.1`;
- frozen Rust consensus and data-format baseline: `0.11.1`;
- the code does not automatically track the latest upstream branch;
- RGB `0.12` contains consensus-level and data-structure changes and is not a compatible upgrade of `rgb11`; future support must use a separate `rgb12` protocol space.

Therefore, “RGB11” in this documentation does not mean every RGB release. It means the frozen version set described above.

### 1.2 Official Rust sources and exact commits

The SAT20 Go implementation uses the following frozen upstream Rust sources as its protocol and interoperability reference. Every reference is pinned to an exact commit; a floating `master`, `main`, or semver range is not an acceptable release baseline.

| Area | Upstream version | Upstream source |
| --- | --- | --- |
| RGB consensus, operation IDs, seals, commitments | `rgb-consensus 0.11.1` | [`rgb-protocol/rgb-consensus@32a67862`](https://github.com/rgb-protocol/rgb-consensus/commit/32a67862aef0f4c7a1fdc7834a3555d702f1bdf0) |
| Operations, consignments, invoicing | `rgb-ops` / `rgb-invoicing 0.11.1` | [`rgb-protocol/rgb-ops@8bcdbf2f`](https://github.com/rgb-protocol/rgb-ops/commit/8bcdbf2fd706782948a4cbd799639709a4ed10d9) |
| PSBT utilities and API | `rgb-psbt-utils 0.11.1` | [`rgb-protocol/rgb-api@102f4b09`](https://github.com/rgb-protocol/rgb-api/commit/102f4b09efa8f0e5987e60714364ba693c89def7) |
| Official NIA, IFA, CFA, and UDA schemata | `rgb-schemas 0.11.1` | [`rgb-protocol/rgb-schemas@906817a0`](https://github.com/rgb-protocol/rgb-schemas/commit/906817a0c24a6bb7d2e53ceac9e2c185ec70a5ff) |
| Strict Encoding | `rgb-strict-encoding 1.0.4` | [`rgb-protocol/rgb-strict-encoding@aa90bf35`](https://github.com/rgb-protocol/rgb-strict-encoding/commit/aa90bf353f53e8220aaacec4a671545808da05ae) |
| Strict Types | `rgb-strict-types 1.0.4` | [`rgb-protocol/rgb-strict-types@9eb3b484`](https://github.com/rgb-protocol/rgb-strict-types/commit/9eb3b484f23b01c70f2a940bda07a207377b3864) |

### 1.3 Wallet interoperability references

The main external wallet oracle is:

- [`RGB-Tools/rgb-lib`](https://github.com/RGB-Tools/rgb-lib), version `0.3.0-beta.7`, pinned to commit [`538f2abaa67d7ce96be32d94092e8f1b9e3ea38e`](https://github.com/RGB-Tools/rgb-lib/commit/538f2abaa67d7ce96be32d94092e8f1b9e3ea38e). It is used to verify wallet state, Esplora synchronization, invoices, consignments, signing, receiving, and balance flows.
- The official [`RGB-WG/rgb`](https://github.com/RGB-WG/rgb) command-line wallet, pinned to tag `v0.11.1-alpha.3` and commit [`a9bba35ceed7e0c4bc4e477f663ab022d7b0a23e`](https://github.com/RGB-WG/rgb/commit/a9bba35ceed7e0c4bc4e477f663ab022d7b0a23e), is used only for manual checks of the CLI surface and wallet derivation paths.

`RGB-WG/rgb v0.11.1-alpha.3` uses alpha.3 crate formats and cannot serve as the `0.11.1` stable consignment/parser release gate. `rgb-lib 0.3.0-beta.7` still depends on rc.11 crates; its prior bidirectional wallet interoperability results are historical compatibility evidence. The stable protocol gate uses pinned `0.11.1` Rust crates.

### 1.4 SAT20 Go implementation

SAT20 uses an independent Go implementation:

- source repository: [`sat20-labs/rgb11`](https://github.com/sat20-labs/rgb11);
- Wallet SDK adapter: [`sat20-labs/sat20wallet`](https://github.com/sat20-labs/sat20wallet/tree/main/sdk/wallet/rgb11);
- exact upstream versions, commits, crate checksums, and translation map: [`UPSTREAM_MANIFEST.json`](https://github.com/sat20-labs/rgb11/blob/main/UPSTREAM_MANIFEST.json);
- official interoperability notes: [`OFFICIAL_INTEROP.md`](https://github.com/sat20-labs/rgb11/blob/main/OFFICIAL_INTEROP.md).

`github.com/sat20-labs/rgb11` is an **independent Go reimplementation** of the frozen Rust baseline. It is not an official upstream RGB Go SDK and is not a Rust library exposed through Go FFI. Consensus structures, Strict Encoding, IDs, seals, anchors, consignments, and PSBT fields must match the frozen upstream implementation. Wallet SDK, DKVS, Indexer, and PWA integration belong to the SAT20 adapter layer.

The manifest records file-level translation relationships, including:

```text
rgb-strict-encoding/rust/src/traits.rs
  -> strict_encoding/encoder.go, strict_encoding/decoder.go

rgb-consensus/src/commit_verify/digest.rs
  -> consensus/tagged_hash.go

rgb-consensus/src/operation/commit.rs
  -> consensus/id.go

rgb-consensus/src/seals/txout/blind.rs
  -> seals/blind.go

rgb-ops/src/containers/consignment.rs
  -> consignment/armor.go
```

A Go capability is considered compatible only after it passes the frozen Rust differential vectors, official parser round trips, and wallet interoperability gates.

## 2. Asset Identity

RGB11 assets have two distinct identity layers:

- **Contract ID / official asset ID**: the protocol-level unique identity and the final validation authority;
- **SAT20 AssetName**: the readable index name used by wallets, UIs, and asset lists.

For newly issued or imported contracts, the current Wallet SDK derives a deterministic AssetName:

```text
rgb11:<type>:<normalized_ticker>@<contract_fingerprint>
```

Example:

```text
rgb11:f:usdt@k7m3q9x2
```

Where:

- `rgb11` is the protocol name;
- `f` represents a fungible asset;
- ticker metadata is lower-cased and restricted in characters and length;
- the default fingerprint is a deterministic 8-character digest of the Contract ID; on a prefix collision, the registry may extend it by 2 characters at a time, up to 16;
- the full Contract ID remains in ticker extension metadata and must not be inferred from the short ticker alone.

A short ticker is a display alias only. Until a primary-asset registry or issuer verification has completed, the UI should retain the fingerprint. A short label such as `usdt` may become the primary display name only after explicit verification.

## 3. Wallet SDK Boundary

The outer `wallet.Manager` exposes RGB11 domain APIs only. The internal `rgb11Manager` owns protocol behavior, including:

- contract issuance, import, and registration;
- consignment decoding and client validation;
- RGB11 UTXO, allocation-proof, and balance projection;
- invoices, address receive capabilities, and transfer preparation;
- PSBT construction, Tapret carrier signing, and transaction broadcast;
- relay, ACK/NACK, and pending-transfer lifecycle;
- encrypted DKVS backup, restore, and multi-device conflict detection;
- RGB11 UTXO lock maintenance and reconstruction.

The outer wallet must not duplicate RGB11 internals or directly manipulate the RGB11 engine store, projection store, or DKVS transport.

## 4. Issuance and Import

### 4.1 Issuance

`IssueRGB11Asset` builds an RGB11 contract from an issuance request and performs the following steps:

1. Select Bitcoin L1 carrier UTXOs;
2. Build allocations and contract state;
3. Generate and validate the RGB11 contract;
4. Store the contract, proofs, and ticker extension metadata;
5. Lock the UTXOs carrying RGB11 state;
6. Update the local asset projection and backup status.

The first release exposes issuance for NIA, IFA, and UDA. CFA contracts can be imported and validated against the frozen official schema, but the current SDK/PWA does not expose CFA issuance.

Balances before and after issuance come from client-verifiable state. They are not synthesized from an ordinary L1 Indexer ticker endpoint.

### 4.2 Import

A contract file must be parsed and validated before the SDK registers:

- Contract ID;
- schema;
- canonical AssetName;
- original ticker, normalized ticker, and fingerprint;
- issuer and control metadata;
- optional reject-list or policy-adapter metadata;
- current validation status.

Invalid, damaged, or conflicting contracts do not enter the usable asset list.

## 5. Receive Modes

### 5.1 Witness Invoice

Witness receive mode uses the fixed P2TR script of the active wallet subaccount. Multiple invoices may therefore have:

- independent `RequestID`, amount, and expiry values;
- the same witness script for the active subaccount;
- independent consignments and validation outcomes.

A fixed address improves passive receiving and address stability. It does not reduce the requirement to validate the Contract ID, seal, allocation, and Bitcoin witness evidence.

### 5.2 Blind Seal

Blind mode uses a single-use seal. The SDK reserves a carrier UTXO and locks it with the `pending-rgb` reason until the receive flow completes, fails, is canceled, or expires.

### 5.3 Configured Address Receive

An application may publish an RGB11 receive capability/profile for a Bitcoin address. A sender resolves that capability and delivers an encrypted consignment through a DKVS mailbox. This supports passive receive flows without requiring the receiver to be online to create a one-time invoice.

An address profile describes capability and delivery location only. It does not mean the receiver has accepted the asset. Acceptance still requires local consignment validation and an ACK.

## 6. Send, Delivery, and ACK

The standard send sequence is:

1. Parse an invoice or address capability;
2. Check asset identity, balance, UTXO locks, and minimum confirmations;
3. Build the transition, consignment, PSBT, and change seals;
4. Persist a pending transfer;
5. Deliver the consignment through the selected transport;
6. Broadcast the Bitcoin transaction according to that transport's durability and ACK policy;
7. Record an ACK or NACK after receiver validation;
8. Track confirmations and update allocations, balance, and UTXO locks.

The SDK supports three transport families:

- configured SAT20 addresses with encrypted DKVS mailbox delivery;
- standard RGB JSON-RPC proxy.
- application-carried out-of-band consignments.

RGB11 no longer uses DKVS `/tmp` relay/ACK records or SAT20-private invoice query parameters. Mailbox records carry encrypted delivery data only; private seal disclosures, complete local consignments, signed transactions, and change seals never enter account backup or the wallet head.

An ACK is not an asset-validity proof. A receiver may issue an ACK only after successful local client validation. The sender must also verify that the ACK is bound to the expected transfer, recipient, and selected transport record.

## 7. UTXO and Balance Model

RGB11 state is bound to Bitcoin UTXOs. Wallet SDK uses two lock reasons:

```text
rgb          confirmed RGB11 state carrier
pending-rgb  UTXO participating in an unfinished receive or send flow
```

On wallet startup, wallet switch, subaccount switch, or snapshot restore, the SDK rebuilds the lock set from the projection store and allocation proofs.

An RGB11 balance is the sum of locally valid allocations. The following do not become spendable balance:

- missing allocation proof;
- failed consignment validation;
- unresolved witness transaction or outpoint;
- Contract ID, schema, or assignment mismatch;
- state rejected by a configured reject list or policy adapter;
- local RGB11 state marked inconsistent or broken.

## 8. DKVS Wallet Backup

RGB11 uses an independent, stable DKVS prefix and does not mix its business keys with account-management or other modules. The core objects are:

```text
/personal/<account_id>/rgb11/<wallet_id>/head
/blob/<account_id>/<rgb11_snapshot_key>
```

The exact keys are generated by `RGB11WalletHeadPath` and `RGB11WalletSnapshotBlobKey`.

### 8.1 Head and Snapshot

- The snapshot contains RGB11 engine records, projection records, and ticker metadata;
- the snapshot is encrypted to the active wallet public key before it enters DKVS;
- the head contains wallet ID, sequence, state hash, and operation ID;
- head and snapshot are written atomically to the target node through one `dkvsManager` key-ETag batch CAS;
- restore validates the head, decrypts the snapshot, and verifies state hash, wallet ID, account index, and engine build ID.

DKVS provides durable storage and synchronization for encrypted state. It does not replace RGB11 asset validation.

### 8.2 Retention Modes

Policy selection uses this order:

1. If the wallet has an active DKVS AUTOPAY delegate, use paid relayable storage;
2. if AUTOPAY is unavailable or its lookup fails, but the current endpoint explicitly enables FREE_LOCAL, fall back to temporary local storage;
3. if neither mode is available, return a storage-policy error.

`FREE_LOCAL` has strict semantics:

- it exists only on the current endpoint;
- it is not relayed through P2P;
- it is excluded from network PathMeta, checkpoints, and snapshots;
- restore is possible only through the same endpoint;
- the UI must not label it as a network-wide backup.

For cross-node persistence, the wallet reads the current `Seq/ETag`, creates a `Seq+1` AUTOPAY record, and sends an ordinary Put. There is no promote API, and a propagated AUTOPAY/PAID key cannot be downgraded to FREE_LOCAL.

### 8.3 Multi-Device Conflicts

One RGB11 wallet supports one active writer at a time. Another device may read or restore, but it must synchronize to the latest head before writing.

If two devices modify state from the same old head, the later submission receives a head conflict or DKVS write conflict. The SDK does not merge two RGB11 states automatically. The application must read the latest head, re-execute the uncommitted operation, and create a new RequestID and signed mutation.

## 9. Security Boundary

RGB11 wallet integration depends on independent checks at each layer:

- RGB11 contract and schema validation;
- consignment integrity and state-transition validation;
- seal and allocation-proof validation;
- Bitcoin transaction, outpoint, script, and confirmation evidence;
- correct wallet signing of PSBTs and Tapret carriers;
- DKVS record identity, signature, sequence, key ETag, and fee proof;
- relay/ACK binding to transfer ID, recipient, and txid/vout.

Failure at one layer cannot be bypassed by an index result or response from another layer.

The current implementation does not define general issuer freezing as an RGB11 consensus rule. An optional reject list or policy adapter affects whether the current wallet accepts a state; its governance and freeze semantics must be defined by the specific asset issuance scheme.

## 10. Current Test Coverage

Wallet SDK includes local real-three-node E2E coverage for:

- fixed-address witness invoices;
- independent RequestIDs;
- atomic RGB11 head plus encrypted-snapshot storage;
- recovery on a new device connected to the same endpoint;
- FREE_LOCAL isolation across endpoints;
- FREE_LOCAL fallback when AUTOPAY lookup fails;
- stale-writer and head-conflict handling;
- invalid amount and receive-mode boundaries;
- existing issuance, transfer, proxy, address delivery, ACK, and allocation unit/integration tests.

These tests use local SatoshiNet bootstrap, core, and miner nodes. They do not depend on a public testnet or an external RGB regtest service.

The Go engine repository retains stable Rust/Go differential vectors and parser round trips, plus historical `rgb-lib` rc.11 bidirectional file-exchange/regtest interoperability evidence. The exact gates and evidence locations are documented in `UPSTREAM_MANIFEST.json` and `OFFICIAL_INTEROP.md`.

## 11. Main Wallet SDK APIs

Common domain entry points include:

```text
IssueRGB11Asset
ImportRGB11Contract / ImportRGB11ContractFile
CreateRGB11Invoice
PrepareRGB11Transfer
PrepareConfiguredRGB11AddressTransfer
DeliverAndBroadcastConfiguredRGB11AddressTransfer
AcceptRGB11Consignment
ValidateRGB11Consignment
RefreshRGB11State
GetRGB11State
GetRGB11AssetBalance
ListRGB11Outputs
SyncRGB11WalletState
RestoreLatestRGB11WalletState
ActivateRGB11WalletState
RebuildRGB11Locks
```

Applications should call these domain APIs instead of constructing DKVS keys, records, sequences, ETags, subscription cursors, or RGB11 internal storage objects directly.
