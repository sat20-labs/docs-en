# STP Messages and Data Model

This document defines the protocol messages, shared data objects, and error semantics a third-party STP client needs to understand. It is written for wallets, SDKs, PWA adapters, CLIs, backend services, and AI Agent tools, so clients in any programming language can connect to a compatible Core Node.

This is not a codebase-specific type reference. The open-source wallet SDK wire message names are useful interoperability references, but implementations should follow the protocol semantics, signatures, state transitions, and verifiable evidence described here.

## SDK Source References

STP client / peer message definitions have moved into the open-source wallet SDK. This document describes the protocol-level structure; the source files are the direct reference for developers who want to verify field names and current wire definitions:

| Content | GitHub |
| --- | --- |
| STP channel messages | [sat20wallet/sdk/wire/types_stp_channel.go](https://github.com/sat20-labs/sat20wallet/blob/main/sdk/wire/types_stp_channel.go) |
| Ping, ActionSync, PerformAction | [sat20wallet/sdk/wire/types.go](https://github.com/sat20-labs/sat20wallet/blob/main/sdk/wire/types.go) |
| `MsgHeader`, `BaseResp` | [sat20wallet/sdk/wire/types_contract.go](https://github.com/sat20-labs/sat20wallet/blob/main/sdk/wire/types_contract.go) |

Cross-language implementations should treat JSON field names as the interoperability boundary. Binary fields are encoded by the current Go SDK according to Go `[]byte` JSON rules; non-Go adapters should standardize hex or base64 at the adapter boundary and keep signing serialization deterministic.

## Participants

| Participant | Protocol Responsibility |
| --- | --- |
| Client Wallet | The user wallet or wallet-authorized adapter. It holds user keys, channel state, commitment transactions, revocation material, and pending operations |
| Core Node | The node that provides STP service to ordinary users. It opens private channels with Client Wallets, co-signs commitment transactions, maintains service-side channel state, and runs or configures an L1 indexer |
| Bootstrap Node | Core Node discovery, admission, and network organization. It is not the ordinary STP peer for a user opening a private channel |
| L1 Indexer | Bitcoin L1 asset fact layer. It indexes UTXOs, Ordinals, ORDX, BRC20, Runes, confirmations, transaction visibility, and reorgs |
| L2 Indexer | SatoshiNet asset fact layer. It indexes SatoshiNet UTXOs, ascend, descend, channels, contracts, and transaction confirmations |

Ordinary STP messages mainly flow between the Client Wallet and the Core Node. L1/L2 indexers do not sign channel state, but they provide evidence clients need to verify asset facts and recover pending operations.

## Protocol Layers

An STP client is usually implemented in three layers:

| Layer | Content |
| --- | --- |
| User operation layer | JSON operations such as `stp.open`, `stp.splicing_in`, `stp.unlock`, `stp.lock`, and `stp.close`. This layer serves UI, CLI, and AI Agents |
| STP peer message layer | Authenticated messages between Client Wallet and Core Node. This layer handles negotiation, signatures, revocation, ack, and state transition |
| Chain and indexer evidence layer | UTXOs, assets, transactions, ascend, descend, and channel state from Bitcoin L1, SatoshiNet, L1 indexer, and L2 indexer |

The upper adapter can hide asset UTXOs, fee UTXOs, BRC20 transfer inscriptions, stub UTXOs, and transaction package details, but it must not hide safety evidence. An Agent should be able to read safety snapshot, commitment export, punish status, force close plan, and transaction status.

## Message Envelope

Every message that changes channel state, spends assets, reveals revocation material, or confirms broadcast results must be authenticated. A recommended envelope is:

| Field | Meaning |
| --- | --- |
| `version` | Protocol message version |
| `msg_id` | Message type or path |
| `chain` | Network identifier, such as mainnet, testnet, or testnet4 |
| `channel_id` | Channel address or channel identifier |
| `commit_height` | Current commitment height; required for messages that change commitment state |
| `request_id` | Stable transaction ID for retry and recovery |
| `sender_pubkey` | Sender public key |
| `node_id` | Core Node identity, optional but recommended |
| `payload` | Message-specific content |
| `signature` | Signature over deterministic serialization excluding the signature field |

The receiver must verify signature, chain ID, channel identity, sender identity, commitment height, and asset amounts. Commitment-height rollback, chain mismatch, invalid signature, asset imbalance, or mismatch with the pending operation must be rejected.

## Shared Data Objects

### Channel Identity

| Field | Meaning |
| --- | --- |
| `channel_id` | Channel identifier, usually represented by the channel address |
| `channel_address` | 2-of-2 address controlled by Client Wallet and Core Node |
| `client_pubkey` | User channel public key |
| `core_node_pubkey` | Core Node channel public key |
| `channel_point` | Current Bitcoin L1 outpoint spent by the commitment transaction |
| `commit_height` | Current commitment height |
| `csv_delay` | Punishment window after force close |

### Asset Descriptor

| Field | Meaning |
| --- | --- |
| `asset_type` | `sats`, `ordx`, `runes`, `brc20`, and similar types |
| `asset_name` | Protocol asset name |
| `amount` | Asset amount as a string to avoid precision loss |
| `binding_sat` | Binding parameter for ORDX-like sat-bound assets |
| `selected_asset_only` | If one UTXO carries multiple assets, this operation only handles the explicitly selected asset |

### UTXO Reference

| Field | Meaning |
| --- | --- |
| `outpoint` | `txid:vout` |
| `address` | Owner address |
| `value_sats` | Plain sats amount; SatoshiNet L2 can be zero |
| `assets` | Assets carried by the output |
| `spendable` | Whether this output can already be used as a transaction input |
| `source` | `l1` or `l2` |

### Commitment Descriptor

| Field | Meaning |
| --- | --- |
| `commitment_txid` | Commitment transaction ID |
| `commitment_tx` | Transaction hex or structured read-only object |
| `commit_height` | Commitment height represented by this transaction |
| `owner` | `local` or `remote` |
| `channel_point` | Channel outpoint spent by this commitment |
| `balances` | Asset ownership under this commitment |
| `deanchor_tx` | Corresponding L2 descend / de-anchor transaction, if any |
| `prev_txs` / `next_txs` | Related transactions for BRC20, stubs, or transaction packages |
| `others` | Additional commitment or sweep transaction used to clean user assets from the channel address |

The client must not persist balances only. After every commitment-height update, it must persist the latest local commitment, remote commitment, revocation material, related transactions, and pending operation.

### Revocation Descriptor

| Field | Meaning |
| --- | --- |
| `revocation_hash` | Revocation commitment for an old state |
| `revocation_secret` | Revocation secret or controlled reference; must not be exposed to unauthorized upper layers |
| `next_commitment_point` | Next commitment point |
| `revoked_commitment_txid` | Revoked remote commitment |
| `punish_material_status` | `missing`, `verified`, `broadcastable`, or `broadcasted` |

### Safety Snapshot

| Field | Meaning |
| --- | --- |
| `status` | `READY_SAFE`, `READY_DEGRADED`, `PUNISH_COVERAGE_UNKNOWN`, and similar states |
| `channel_id` | Channel identifier |
| `commit_height` | Current commitment height |
| `channel_point` | Current channel outpoint |
| `local_commitment` | Summary of the latest exit transaction the local side can broadcast |
| `remote_commitments` | Peer commitment summaries and revocation coverage |
| `punish_coverage` | Punishment coverage for old remote commitments |
| `pending` | Pending operation |
| `l1_view` / `l2_view` | Indexer evidence summary |

## Message Structure Index

This section lists the main message structures a third-party client needs to implement or understand. Types are described at the protocol level: `bytes` means binary data, `bytes[]` means an array of binary values, `bytes[][]` means a two-dimensional signature array, and `string[]` means an array of strings.

For readability, this document groups embedded fields as `CommitSigInfo`, `SplicingSigInfo`, or `RevokeAndAck`. On the actual JSON wire, these fields are expanded according to the SDK definitions. Third-party implementations should keep field names and signature serialization compatible.

### Common Structures

`MsgHeader`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` | int | Message version |
| `msgId` | string | Message type or path |

`BaseResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` | int | `0` means success; non-zero means failure |
| `msg` | string | Error or status text |

`OpenChannelFee`

| Field | Type | Meaning |
| --- | --- | --- |
| `manageFee` | int64 | Channel management fee |
| `mortgageFee` | int64 | Core Node admission or service-related fee; ordinary client open should not treat this as user staking |
| `minReserveSats` | int64 | Minimum plain sats reserved for the channel |
| `commitmentFee` | int64 | Commitment transaction fee budget |
| `commitmentFeeRate` | int64 | Commitment fee-rate parameter |
| `splicingInFee` | int64 | Splicing-in service fee |
| `splicingOutFee` | int64 | Splicing-out service fee |

`CommitSigInfo`

| Field | Type | Meaning |
| --- | --- | --- |
| `commitSig` | bytes[] | Commitment signatures |
| `commitDeAnchorSig` | bytes[] | De-anchor / anchor signatures for the commitment |
| `commitPrevTxSig` | bytes[][] | Signatures for previous transactions, such as BRC20 transfer-inscription transactions |
| `commitNextTxSig` | bytes[][] | Signatures for next transactions |
| `commitOtherPrevTxSig` | bytes[][][] | Previous-transaction signatures for additional sweep transactions |
| `commitOtherTxSig` | bytes[][] | Additional sweep transaction signatures |

`SplicingSigInfo`

| Field | Type | Meaning |
| --- | --- | --- |
| `splicingPrevTxSig` | bytes[][] | Signatures for splicing previous transactions |
| `splicingSig` | bytes[] | Splicing transaction signatures |
| `deAnchorSig` | bytes[] | De-anchor or anchor transaction signatures |

`RevokeAndAck`

| Field | Type | Meaning |
| --- | --- | --- |
| `revocation` | bytes32 | Preimage to the revocation hash of the prior commitment |
| `nextRevKey` | bytes | Next commitment point or next revocation key |

### Connection and Sync Structures

`AbbrChannelInfo`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` | int | Channel data version |
| `channelId` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `staticMerkleRoot` | bytes | Static channel data root |
| `localAssetMerkleRoot` | bytes | Local asset root |
| `remoteAssetMerkleRoot` | bytes | Remote asset root |

`PingRequest`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `pubKey` | bytes | Sender public key |
| `mode` | string | Ping mode |
| `info` | AbbrChannelInfo | Channel summary, optional |
| `nodeId` | bytes | Core Node identity, optional |

`PingReq = PingRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `msgSig` | bytes | Signature over message content |

`PingResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `commitHeight` | int | Commitment height seen by the peer |
| `action` | string | Next action |
| `param` | string | Next action parameter |
| `paramSig` | bytes | Parameter signature |

`ActionSyncReq = ActionSyncRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `pubKey` | bytes | Requester public key |
| `reason` | string | Sync reason |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`ActionSyncResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `channelData` | bytes | Recoverable channel data |

### Open Structures

`ChannelOpenReq = OpenChannelRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `nodeId` | bytes | Core Node identity |
| `channelType` | int | Channel type |
| `channelWalletId` | int | Sub-wallet or sub-account ID |
| `fundingKey` | bytes | Local funding public key |
| `feeRate` | int64 | Bitcoin L1 fee-rate parameter |
| `localFundingAmount` | int64 | Local funding amount in sats |
| `outpoints` | string[] | Optional funding inputs |
| `needSendFundingTx` | bool | Whether the client needs to build and broadcast the funding transaction |
| `skipOpeningAnchorTx` | bool | Whether to skip opening anchor in a recovery path |
| `l2DrainTxId` | string | Related L2 drain txid in a recovery path |
| `memo` | bytes | Memo |
| `msgSig` | bytes | Request signature |

`ChannelOpenResp = BaseResp + AcceptChannel`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Open reservation ID |
| `commitHeight` | int | Initial commitment height |
| `csv` | uint16 | CSV delay |
| `openFee` | OpenChannelFee | Open fee parameters |
| `fundingKey` | bytes | Core Node funding public key |
| `revbasePoint` | bytes | Core Node revocation base point |
| `commitPoint` | bytes | Core Node commitment point |
| `invoiceSig` | bytes | Server invoice signature |

`FundingCreatedReq = FundingCreated + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Open reservation ID |
| `fundingPoint` | string | Funding outpoint |
| `revbasePoint` | bytes | Client revocation base point |
| `commitPoint` | bytes | Client commitment point |
| `commitSig` | bytes[] | Client initial commitment signatures |
| `deAnchorSig` | bytes[] | Client de-anchor signatures |
| `msgSig` | bytes | Request signature |

`FundingCreatedResp = BaseResp + FundingSigned`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Open reservation ID |
| `commitSig` | bytes[] | Core Node commitment signatures |
| `deAnchorSig` | bytes[] | Core Node de-anchor signatures |

`FundingBroadcastedReq = FundingBroadcasted + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Open reservation ID |
| `fundingTxId` | string | Broadcast Bitcoin L1 funding txid |
| `msgSig` | bytes | Request signature |

`FundingBroadcastedResp = BaseResp`

### Unlock Structures

`UnlockReq = UnlockRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `channel` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `assetName` | string | Asset name |
| `amt` | string[] | Released amount list |
| `feeRate` | int64 | Compatibility field; SatoshiNet unlock should not require user-supplied Bitcoin fee rate |
| `feeUtxos` | string[] | Compatibility field; adapters should not expose this for ordinary Agent selection |
| `address` | string[] | Target L2 address list |
| `memo` | bytes | Memo |
| `reason` | string | Operation reason |
| `more` | bytes | Extension data |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`UnlockResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `id` | int64 | Reservation ID |
| `revealKey` | bytes | Reveal key for this round |
| `rev` | bytes | Revocation key for this round |
| `nextRevKey` | bytes | Next revocation key |
| `feeRate` | int64 | Returned fee parameter |

`UnlockCommitSigReq = CommitSigInfo + rev keys + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `commitSigInfo` | CommitSigInfo | New commitment signature set |
| `rev` | bytes | Local revocation key |
| `nextRevKey` | bytes | Local next revocation key |
| `msgSig` | bytes | Request signature |

`UnlockCommitSigResp = BaseResp + CommitSigInfo + RevokeAndAck`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Reservation ID |
| `commitSigInfo` | CommitSigInfo | Peer commitment signature set |
| `rev` | RevokeAndAck | Peer revoke-and-ack for the old state |

`UnlockRevokeAndAckReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `rev` | RevokeAndAck | Client revoke-and-ack for the old state |
| `unlockSig` | bytes[] | Unlock transaction signatures |
| `msgSig` | bytes | Request signature |

`UnlockRevokeAndAckResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `id` | int64 | Reservation ID |
| `unlockSig` | bytes[] | Core Node unlock transaction signatures |

### Lock Structures

`LockReq = LockRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `channel` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `assetName` | string | Asset name |
| `amt` | string | Locked amount |
| `feeRate` | int64 | Compatibility field |
| `lockUtxos` | string[] | Compatibility field; adapter usually selects internally |
| `feeUtxos` | string[] | Compatibility field; adapter usually selects internally |
| `revealKey` | bytes | Reveal key |
| `rev` | bytes | Revocation key |
| `nextRevKey` | bytes | Next revocation key |
| `needSendLockTx` | bool | Whether a lock transaction needs to be sent |
| `memo` | bytes | Memo |
| `reason` | string | Operation reason |
| `more` | bytes | Extension data |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`LockResp = BaseResp + CommitSigInfo + rev keys`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Reservation ID |
| `feeRate` | int64 | Returned fee parameter |
| `commitSigInfo` | CommitSigInfo | Core Node commitment signature set |
| `rev` | bytes | Core Node revocation key |
| `nextRevKey` | bytes | Core Node next revocation key |

`LockCommitSigAndRevokeReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `commitSigInfo` | CommitSigInfo | Client commitment signature set |
| `rev` | RevokeAndAck | Client revoke-and-ack for the old state |
| `msgSig` | bytes | Request signature |

`LockCommitSigAndRevokeResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `id` | int64 | Reservation ID |
| `rev` | RevokeAndAck | Core Node revoke-and-ack for the old state |
| `lockSig` | bytes[] | Core Node lock transaction signatures |

`LockAckReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `lockSig` | bytes[] | Client lock transaction signatures |
| `msgSig` | bytes | Request signature |

`LockAckResp = BaseResp + id`

### Splicing-In Structures

`SplicingInReq = SplicingInRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `channel` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `assetName` | string | Asset name |
| `amt` | string | Amount entering the channel |
| `stub` | string | Stub UTXO, optional |
| `utxos` | string[] | Compatibility field; adapter usually selects asset UTXOs internally |
| `fees` | string[] | Compatibility field; adapter usually selects fee UTXOs internally |
| `preTxInputs` | string[] | Previous transaction inputs |
| `revealKey` | bytes | Reveal key |
| `needSendSplicingTx` | bool | Whether an L1 splicing transaction needs to be built and broadcast |
| `feeRate` | int64 | Bitcoin L1 fee-rate parameter |
| `reason` | string | Operation reason |
| `memo` | bytes | Memo |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`SplicingInResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `id` | int64 | Reservation ID |
| `serviceFee` | int64 | Service fee |
| `rev` | bytes | Core Node revocation key |
| `nextRevKey` | bytes | Core Node next revocation key |
| `newCapacity` | int64 | New channel capacity |
| `newLocalBalance` | string | New local balance |
| `newRemoteBalance` | string | New remote balance |
| `invoiceSig` | bytes | Invoice signature |

`SplicingInCommitSigReq = SplicingSigInfo + CommitSigInfo + rev keys + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `splicingSigInfo` | SplicingSigInfo | Splicing and anchor/de-anchor signatures |
| `commitSigInfo` | CommitSigInfo | Commitment signatures |
| `rev` | bytes | Client revocation key |
| `nextRevKey` | bytes | Client next revocation key |
| `msgSig` | bytes | Request signature |

`SplicingInCommitSigResp = BaseResp + SplicingSigInfo + CommitSigInfo + RevokeAndAck`

`SplicingInRevokeAndAckReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `txId` | string | Splicing txid |
| `rev` | RevokeAndAck | Client revoke-and-ack for the old state |
| `msgSig` | bytes | Request signature |

`SplicingInRevokeAndAckResp = BaseResp + id`

### Splicing-Out Structures

`SplicingOutReq = SplicingOutRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `channel` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `assetName` | string | Asset name |
| `amt` | string | Exit amount |
| `stub` | string | Stub UTXO |
| `utxos` | string[] | Compatibility field; adapter usually selects asset UTXOs internally |
| `fees` | string[] | Compatibility field; adapter usually selects fee UTXOs internally |
| `preTxInputs` | string[] | Previous transaction inputs |
| `revealKey` | bytes | Reveal key |
| `address` | string | Bitcoin L1 destination address |
| `feeRate` | int64 | Bitcoin L1 fee-rate parameter |
| `reason` | string | Operation reason |
| `memo` | bytes | Memo |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`SplicingOutResp` has the same fields as `SplicingInResp`.

`SplicingOutCommitSigReq = SplicingSigInfo + CommitSigInfo + rev keys + msgSig`

`SplicingOutCommitSigResp = BaseResp + SplicingSigInfo + CommitSigInfo + RevokeAndAck`

`SplicingOutRevokeAndAckReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `channel` | string | Channel identifier |
| `id` | int64 | Reservation ID |
| `deAnchorTxId` | string | L2 descend / de-anchor txid |
| `rev` | RevokeAndAck | Client revoke-and-ack for the old state |
| `msgSig` | bytes | Request signature |

`SplicingOutRevokeAndAckResp = BaseResp + id`

### Close Structures

`ChannelCloseReq = CloseChannelRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `channel` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `feeRate` | int64 | Bitcoin L1 fee-rate parameter, optional |
| `revealKey` | bytes | Reveal key, optional |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`ChannelCloseResp = BaseResp + ClosingSigned`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Reservation ID |
| `deAnchorSig` | bytes[] | De-anchor signatures |
| `closingsig` | bytes[] | Closing transaction signatures |
| `prevTxSig` | bytes[][] | Previous transaction signatures |

`ClosingSignedReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Reservation ID |
| `deAnchorSig` | bytes[] | Client de-anchor signatures |
| `closingsig` | bytes[] | Client closing signatures |
| `prevTxSig` | bytes[][] | Client previous transaction signatures |
| `channel` | string | Channel identifier |
| `msgSig` | bytes | Request signature |

`ClosingSignedResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `splicingTxId` | string | Exit or splicing txid produced during close |

`ClosingBroadcastedReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Reservation ID |
| `deAnchorTxId` | string | Broadcast de-anchor txid |
| `channel` | string | Channel identifier |
| `msgSig` | bytes | Request signature |

`ClosingBroadcastedResp = BaseResp`

### Recover Payment Structures

`RecoverPaymentRequireReq = RecoverPaymentRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `channel` | string | Channel identifier |
| `commitHeight` | int | Current commitment height |
| `paymentTxId` | string | Payment txid to recover |
| `reason` | string | Recovery reason |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

The later recover-payment structures follow the same commit-sig and revoke-and-ack pattern as unlock, with the final signature field named `paymentSig`.

### PerformAction Structures

`PerformActionReq = PerformActionRequest + msgSig`

| Field | Type | Meaning |
| --- | --- | --- |
| `version` / `msgId` | MsgHeader | Message header |
| `action` | string | Action / reservation type |
| `param` | bytes | Action parameters |
| `feeRate` | int64 | Bitcoin L1 fee-rate parameter |
| `reqTime` | int64 | Request time |
| `sendInL1` | bool | Whether fee or related transaction is sent on L1 |
| `more` | bytes | Extension data |
| `pubKey` | bytes | Requester public key |
| `nodeId` | bytes | Core Node identity, optional |
| `msgSig` | bytes | Request signature |

`PerformActionResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `id` | int64 | Action ID |
| `serviceAddress` | string | Service fee address |
| `serviceFee` | int64 | Service fee |
| `invoice` | bytes | Invoice |
| `invoiceSig` | bytes | Invoice signature |

`PerformActionAckReq`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | int64 | Action ID |
| `tx` | string | Fee transaction hex |
| `txId` | string | Fee txid |

`PerformActionAckResp`

| Field | Type | Meaning |
| --- | --- | --- |
| `code` / `msg` | BaseResp | Result |
| `id` | int64 | Action ID |
| `status` | int | Action status |
| `actionResvId` | int64 | Related reservation ID |
| `actionStatus` | int | Related reservation status |
| `actionResult` | bytes | Action result |

## Message Families

### Connection and Sync

| Message | Direction | Semantics |
| --- | --- | --- |
| `PingRequest` / `PingReq` | Client Wallet -> Core Node | Handshake, identity proof, and channel summary sync |
| `PingResponse` / `PingResp` | Core Node -> Client Wallet | Returns peer commitment height, next action, or sync request |
| `ActionSyncRequest` / `ActionSyncReq` | Either side -> peer | Requests operation-state sync after restart, unknown result, or state mismatch |
| `ActionSyncResp` | Peer -> requester | Returns recoverable channel data or pending-operation data |

### Open Channel

| Message | Direction | Semantics |
| --- | --- | --- |
| `OpenChannelRequest` / `ChannelOpenReq` | Client Wallet -> Core Node | Requests a private channel with funding intent, channel public key, amount, and memo |
| `ChannelOpenResp` | Core Node -> Client Wallet | Accepts the channel and returns CSV, fees, server funding key, revocation base point, and commitment point |
| `FundingCreatedReq` | Client Wallet -> Core Node | Submits funding outpoint, initial commitment signatures, and related de-anchor signatures |
| `FundingCreatedResp` | Core Node -> Client Wallet | Returns server signatures and forms a verifiable initial commitment state |
| `FundingBroadcastedReq` | Client Wallet -> Core Node | Notifies that Bitcoin L1 funding was broadcast |
| `FundingBroadcastedResp` | Core Node -> Client Wallet | Confirms transition into confirmation and anchor tracking |

After open completes, the client must verify funding through the L1 indexer, ascend / anchor through the L2 indexer, and produce a safety snapshot.

### Unlock

| Message | Direction | Semantics |
| --- | --- | --- |
| `UnlockRequest` / `UnlockReq` | Client Wallet -> Core Node | Requests release of channel assets to the user's L2 address |
| `UnlockResp` | Core Node -> Client Wallet | Accepts the update and returns revocation / next-revocation material for this round |
| `UnlockCommitSigReq` | Client Wallet -> Core Node | Sends new commitment signatures and local next-state material |
| `UnlockCommitSigResp` | Core Node -> Client Wallet | Returns server commitment signatures and revoke-and-ack for the old state |
| `UnlockRevokeAndAckReq` | Client Wallet -> Core Node | Client revokes the old state and confirms unlock transaction signatures |
| `UnlockRevokeAndAckResp` | Core Node -> Client Wallet | Server confirms and the round completes |

Unlock advances commit height. SatoshiNet has no Bitcoin L1 fee-rate semantics, and ordinary adapters should not require users to provide fee UTXOs.

### Lock

| Message | Direction | Semantics |
| --- | --- | --- |
| `LockRequest` / `LockReq` | Client Wallet -> Core Node | Requests locking user L2 assets back into channel protection |
| `LockResp` | Core Node -> Client Wallet | Accepts the update and returns commitment-signature material plus next revocation material |
| `LockCommitSigAndRevokeReq` | Client Wallet -> Core Node | Sends commitment signatures and revokes the old state |
| `LockCommitSigAndRevokeResp` | Core Node -> Client Wallet | Returns server revoke-and-ack and lock transaction signatures |
| `LockAckReq` | Client Wallet -> Core Node | Final lock transaction signature confirmation |
| `LockAckResp` | Core Node -> Client Wallet | Round completes |

Lock-with-expand can be understood as a combined Lock and Expand capability: when channel capacity or asset coverage is insufficient, the client should be able to bring assets back under channel control.

### Splicing-In

| Message | Direction | Semantics |
| --- | --- | --- |
| `SplicingInRequest` / `SplicingInReq` | Client Wallet -> Core Node | Requests adding Bitcoin L1 assets to an existing channel |
| `SplicingInResp` | Core Node -> Client Wallet | Returns service fee, new capacity, new balances, and next revocation material |
| `SplicingInCommitSigReq` | Client Wallet -> Core Node | Sends splicing, anchor/de-anchor, and commitment signatures |
| `SplicingInCommitSigResp` | Core Node -> Client Wallet | Returns server signatures and revoke-and-ack |
| `SplicingInRevokeAndAckReq` | Client Wallet -> Core Node | Notifies splicing txid and revokes the old state |
| `SplicingInRevokeAndAckResp` | Core Node -> Client Wallet | Round completes |

Expand reuses splicing-in safety semantics, but the asset is already at the channel address. The client must use L1/L2 indexers to determine whether it has already ascended, avoiding duplicate L2 issuance.

### Splicing-Out

| Message | Direction | Semantics |
| --- | --- | --- |
| `SplicingOutRequest` / `SplicingOutReq` | Client Wallet -> Core Node | Requests exiting channel assets to a Bitcoin L1 address |
| `SplicingOutResp` | Core Node -> Client Wallet | Returns service fee, new capacity, new balances, and next revocation material |
| `SplicingOutCommitSigReq` | Client Wallet -> Core Node | Sends L1 exit, L2 descend / de-anchor, and commitment signatures |
| `SplicingOutCommitSigResp` | Core Node -> Client Wallet | Returns server signatures and revoke-and-ack |
| `SplicingOutRevokeAndAckReq` | Client Wallet -> Core Node | Notifies de-anchor txid and revokes the old state |
| `SplicingOutRevokeAndAckResp` | Core Node -> Client Wallet | Round completes |

After splicing-out, the client must link L2 descend with L1 output through indexer evidence. BRC20 may require a transfer-inscription transaction package. Runes and ORDX must follow their own L1 transfer rules.

### Close

| Message | Direction | Semantics |
| --- | --- | --- |
| `CloseChannelRequest` / `ChannelCloseReq` | Client Wallet -> Core Node | Requests cooperative channel close |
| `ChannelCloseResp` | Core Node -> Client Wallet | Returns closing, de-anchor, and related transaction signatures |
| `ClosingSignedReq` | Client Wallet -> Core Node | Client signs and submits the closing transaction |
| `ClosingSignedResp` | Core Node -> Client Wallet | Returns exit or splicing txid |
| `ClosingBroadcastedReq` | Client Wallet -> Core Node | Notifies that close-related transactions were broadcast |
| `ClosingBroadcastedResp` | Core Node -> Client Wallet | Channel enters closed or waiting-confirmation state |

Force close does not depend on the peer being online. After broadcasting the latest local commitment, the client must construct sweep according to CSV conditions and continue monitoring whether the peer broadcasts an old state.

### Recover Payment

| Message | Direction | Semantics |
| --- | --- | --- |
| `RecoverPaymentRequest` / `RecoverPaymentRequireReq` | Either side -> peer | Requests recovery when a result is unknown or payment state differs |
| `RecoverPaymentRequireResp` | Peer -> requester | Returns revocation / next-revocation material needed for recovery |
| `RecoverPaymentCommitSigReq` | Requester -> peer | Re-submits commitment signatures |
| `RecoverPaymentCommitSigResp` | Peer -> requester | Returns corresponding commitment signatures and revoke-and-ack |
| `RecoverPaymentRevokeAndAckReq` | Requester -> peer | Completes old-state revocation and payment-signature confirmation |
| `RecoverPaymentRevokeAndAckResp` | Peer -> requester | Recovery completes |

### Remote Actions

| Message | Direction | Semantics |
| --- | --- | --- |
| `PerformActionRequest` / `PerformActionReq` | Client Wallet -> Core Node | Requests a Core Node-participating action |
| `PerformActionResp` | Core Node -> Client Wallet | Returns action id, service address, service fee, and invoice |
| `PerformActionAckReq` | Client Wallet -> Core Node | Submits fee transaction or confirmation material |
| `PerformActionAckResp` | Core Node -> Client Wallet | Returns action status and result |

Remote actions do not replace STP channel safety rules. If an action changes channel assets or commitment state, it must still reduce back to commitment transactions, revocation material, commit height, and indexer evidence.

## Errors and Unknown Results

| Condition | Handling Rule |
| --- | --- |
| Invalid signature, wrong chain ID, commit height rollback | Explicit failure; reject the message |
| Asset imbalance, spent UTXO, channel state mismatch | Explicit failure or recovery path; do not continue normal value movement |
| EOF, timeout, connection reset, service restart | Unknown result; do not treat as failure |
| Broadcast response lost | Treat as possibly successful; query L1/L2 transaction visibility and keep inputs locked |
| Indexer has not returned the transaction yet | Query mempool, peer state, and pending operation; wait for convergence |

The client may retry only when both sides are still in the same old safe state, no pending operation exists, and all related L1/L2 transactions remain invisible.

## Testnet Fault Injection

Testnet may expose old-commitment retention, old-commitment broadcast, punish construction, and punish broadcast so AI Agents can verify that they hold the material needed to protect user assets.

These capabilities must satisfy:

1. They are enabled only on testnet.
2. Mainnet interfaces reject them directly.
3. Agents must dry-run verify commitment, de-anchor, punish, and sweep before broadcast.
4. After an old commitment is broadcast, the channel should immediately move to a closed or punished path and must not allow normal value movement.

Mainnet safety does not depend on fault-injection interfaces. Mainnet clients depend on locally persisted commitments, revocation material, watchtower monitoring, and L1/L2 indexer evidence.
