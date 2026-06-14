# Third-Party STP Client Implementation Checklist

This checklist is for wallet teams, SDK teams, and AI Agent adapter builders implementing an STP client. It is not tied to one repository. It is an interoperability checklist: if a client satisfies these requirements, it should be able to connect to a compatible Core Node in any programming language.

## Minimum Viable Client

| Capability | Acceptance standard |
| --- | --- |
| Wallet keys | Can generate or import user keys and sign STP messages and transactions |
| Service discovery | Can obtain Core Node network, endpoint, public key, node id, and feature list |
| Message authentication | Can deterministically serialize, sign, verify, and validate chain ID |
| L1 query | Can query Bitcoin L1 UTXOs, assets, transaction visibility, confirmations, and raw transactions |
| L2 query | Can query SatoshiNet UTXOs, assets, transaction visibility, channel-address state, and ascend / descend records |
| Transaction engine | Can build, verify, sign, and broadcast Bitcoin L1, SatoshiNet, commitment, punishment, and sweep transactions |
| State store | Can persist channel state, commitment height, latest commitment, revoked-state punishment material, and pending operations |
| Recovery | Can recover pending operations after restart without losing channel-control ability |
| Safety interfaces | Can return safety snapshot, commitments, punishment coverage, and force-close plan to an Agent |

A client that can open, unlock, and lock but cannot export safety snapshots, prove punishment coverage, or recover pending operations is not a complete STP client.

## Data That Must Be Persisted

Each channel must persist:

| Data | Purpose |
| --- | --- |
| channel id / channel address | Identifies the channel and 2-of-2 control address |
| client pubkey / Core Node pubkey | Verifies message identity and rebuilds the channel script |
| channel point | Current Bitcoin L1 outpoint spent by commitment transactions |
| funding / splicing UTXO set | Determines which L1 assets are channel-managed |
| commit height | Detects rollback and proves monotonic state progression |
| local commitment tx | Latest exit transaction the user can broadcast |
| remote commitment tx | Peer commitment watched for old-state fraud |
| local / remote balance | Asset ownership in the current commitment state |
| CSV delay | Waiting window before sweeping local force-close outputs |
| revoked remote commitment index | Identifies peer cheating with old states |
| punish tx or construction material | Punishes old remote commitments |
| pending reservation | Recovery entry for unknown or unconfirmed operations |
| related L1 / L2 txids | Lets the client continue polling after restart |

Persisting order matters. Before releasing old-state revocation material, the client must already have stored the new commitment state, latest local commitment, remote commitment, and safety material.

## Agent Safety Interfaces

| Interface | Minimum returned data |
| --- | --- |
| `stp.status` | Network, Core Node status, channel list, channel status, commitment height |
| `stp.safety_snapshot` | Channel point, commitment height, commitment availability, balances, CSV, Merkle roots, punishment coverage, `l2_spendable_balance`, `l2_pending_balance` |
| `stp.commitment_export` | Current local / remote commitment txid, transaction hex or structure, balances, and verification data |
| `stp.punish_status` | Revoked remote commitments, punishment tx list, verified flag, broadcastable flag |
| `stp.punish_build` | Builds and verifies a punishment transaction for a given old commitment without exposing revocation secrets |
| `stp.punish_broadcast` | Broadcasts a verified punishment transaction |
| `stp.force_close_plan` | Current local commitment, CSV delay, and later sweep conditions |
| `stp.sweep_build` | Builds, signs, and verifies a sweep transaction after CSV; supports dry-run and authorized broadcast |

An Agent should only initiate ordinary value movement when `stp.safety_snapshot` returns `READY_SAFE` and punishment coverage is `NO_REVOKED_REMOTE_STATE` or `COVERED`. `PUNISH_COVERAGE_UNKNOWN` and `PUNISH_COVERAGE_MISSING` block splicing, unlock, lock, and close.

`READY_DEGRADED` is read-only. It can mean commitments exist but funding is unconfirmed, peer state is still converging, or the adapter is recovering. It must not authorize unlock, lock, splicing, close, or punish drill.

## Preflight Checks Before Value Movement

Before any post-open value movement, the client must verify:

1. Wallet, Core Node, Bitcoin L1 indexer, and SatoshiNet indexer are on the same network.
2. Channel status is ready and no pending reservation exists.
3. Commitment height has not rolled back; if peer status is available, height differences are explained.
4. Local and remote commitments exist and spend the current channel point.
5. Commitment outputs, asset roots, and local / remote balances match the requested asset movement.
6. Revoked remote commitments have verified or broadcastable punishment coverage, or there are explicitly no revoked remote commitments.
7. Internally selected inputs are not locked by another pending operation.
8. For unlock/lock, the target asset is in spendable L2 UTXOs, not only in pending L2 state.
9. Fee inputs are legal. Except for the internal plain-sat unlock exception, initiator fees are not paid from channel-address UTXOs.
10. Mainnet operations present asset, amount, destination, fee, and action type for user authorization.
11. Pending operation and related txids are persisted before broadcast or final commitment exchange.

## Asset Rules

| Asset | Rule |
| --- | --- |
| Plain sats | Bitcoin L1 has dust and fee constraints. SatoshiNet L2 has no Bitcoin dust constraint |
| ORDX | ORDX binds to sats and must use `bindingSat`; Ordinals NFT objects do not ascend |
| Runes | Runes can be carried by zero-sat SatoshiNet UTXOs. L1 still follows Bitcoin output rules |
| BRC20 | BRC20 can require transfer inscription and commit/reveal packages. The adapter selects or creates transfer UTXOs internally |

If one L1 UTXO carries multiple assets, STP splicing-in only processes the asset explicitly specified by the request. Other assets do not enter SatoshiNet and must not be included in L2 balance expectations.

The Agent-facing splicing-in API hides raw asset inputs. Internal coin selection should avoid multi-asset UTXOs where possible. If one must be used, the preview must explain which asset will ascend and which assets will be ignored.

## Unknown-Result Recovery

The client must treat timeout, connection interruption, and service unavailability as unknown result, not explicit failure.

Unified recovery flow:

1. Stop resubmitting the request and lock related UTXOs.
2. Persist request JSON, reservation, txids, channel id, asset, amount, and error text.
3. Query the pending reservation.
4. Query related Bitcoin L1 and SatoshiNet txids.
5. Query Core Node channel status, commitment height, channel point, and pending state.
6. If a transaction is visible, either side advanced, or the reservation exists, keep polling and recover the original operation.
7. Retry only after both sides remain at the same old safe state, no pending reservation exists, and all related transactions remain invisible.

This rule applies to open, close, splicing-in, splicing-out, unlock, and lock.

## Recovery Acceptance

| Recovery action | Scenario | Acceptance standard |
| --- | --- | --- |
| `stp.restore` | Local state is missing, but peer or backup still has latest channel state | Safety snapshot proves latest commitment and punishment coverage |
| `stp.reopen` | Channel is closed or missing, but ledger proves an existing client-Core channel and the channel address still contains user assets or needs funding | No staking required. If needed, the user wallet creates new L1 funding. Value movement waits for funding confirmation and both sides ready |
| `stp.rebuild` | L1 channel point changed, but SatoshiNet ledger proves anchored assets | Restores channel without duplicate opening anchor |
| `stp.expand` | L1 asset is at the channel address but not covered by current commitment state | Asset enters commitment state and commitment height advances |
| interrupted splicing-in recovery | L1 funding UTXO was already ascended but client/peer did not complete splicing-in | Reuses existing L2 anchor output and does not anchor again |

No-anchor rebuild must rely on the SatoshiNet channel ledger. If the client cannot prove whether an L1 UTXO already corresponds to ascend or descend balance, it should refuse automatic recovery rather than double-issue assets.

## Core Node Interoperability Tests

Before a third-party client connects to a Core Node, it should pass:

| Test | Pass condition |
| --- | --- |
| open | A normal client opens a channel with a Core Node without staking |
| safety snapshot | After open, the client can read local / remote commitments, channel point, commitment height, and CSV |
| sats unlock / lock | Commitment height increases monotonically and balances change correctly |
| Runes splicing-in / unlock / lock | Runes enter the channel and move between L2 personal address and channel |
| BRC20 splicing-in / unlock / lock | BRC20 enters the channel and moves between L2 personal address and channel |
| splicing-out | At least one protocol asset exits to Bitcoin L1 |
| unknown-result recovery | Unknown broadcast or peer-ack result does not double-spend inputs and eventually converges |
| punishment coverage | Every state advance can prove old remote-commitment coverage |
| force-close plan | Peer offline produces broadcastable local commitment and sweep conditions |
| restart recovery | Pending reservation, commitments, and safety snapshot recover after client restart |

## Mainnet Release Gates

A mainnet client must satisfy:

1. Keys and mnemonics always stay inside the wallet boundary, not inside the Agent.
2. Mainnet value movement requires explicit user confirmation.
3. Every channel update can produce a before/after safety snapshot.
4. Database crash or process exit does not lose the latest commitment or pending operation.
5. `PUNISH_COVERAGE_UNKNOWN`, `PUNISH_COVERAGE_MISSING`, commitment rollback, balance mismatch, and invalid signature block new value movement.
6. Testnet fault-injection interfaces are unavailable on mainnet.
7. The Agent can explain asset location, exit path, punishment coverage, and remaining risk.

The user experience can be simple, but the client safety model cannot be simplified. Users may not understand RSMC, commitments, or punishment transactions; the wallet and Agent must understand them and keep proving that the safety conditions hold.
