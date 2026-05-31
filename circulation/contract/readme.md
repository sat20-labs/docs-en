Smart Contracts
====

This document defines the common smart-contract protocol on SatoshiNet. Contract-type specific rules are defined in:

1. [Template Contracts](template.md)
2. [EVM Contracts](evm.md)
3. [Natural Language Contracts](agent.md)


Protocol Boundary
----

A SatoshiNet smart contract is a programmable consensus state machine. Contract state is maintained by SatoshiNet nodes. Real asset sources and asset settlement are represented by the SatoshiNet UTXO model.

Smart contracts are different from channel contracts. Channel contracts are based on RSMC, channel state, commitment transactions, multi-party signatures, and STP workflows. Smart contracts are based on on-chain transactions, contract VM state, canonical `CONTRACT_RESULT`, and block-level state roots. Asset transfers initiated by smart contracts do not require private-key signatures; they must be authorized by VM execution results.


Contract Types
----

The first phase defines three smart-contract types:

1. Template contracts: Go runtimes embedded in SatoshiNet nodes, identified by `ContractTypeTemplate`.
2. EVM contracts: contracts executed by the EVM executor, identified by `ContractTypeEVM`.
3. Natural language contracts: natural language agreements supported by AI Agents, identified by `ContractTypeAgent`. The first phase prioritizes prediction Agent contracts and lets them enter the SatoshiNet contract-result flow in single-CoreNode-Agent mode.


Core Model
----

1. UTXOs are the only source of on-chain assets.
2. The VM is the contract state machine.
3. `CONTRACT_DEPLOY` creates contracts.
4. `CONTRACT_INVOKE` calls contracts.
5. `CONTRACT_RESULT` settles asset transfers produced by the VM.
6. `COINBASE_CONTRACT_STATE_ROOT` commits the post-execution contract state root in coinbase.
7. All nodes must deterministically replay contract execution.
8. Contract UTXOs are not unlocked by traditional private-key signatures. They are spent only when authorized by VM execution results.


Address Rules
----

Mainnet contract address format:

```text
ca + version + type + hash
```

Testnet contract address format:

```text
tc + version + type + hash
```

Fields:

1. `ca` is the mainnet contract address prefix.
2. `tc` is the testnet contract address prefix.
3. `version` is currently `1`.
4. `type` identifies the contract type.
5. `hash` length and calculation are defined by each contract type.


Transaction Types
----

Contract-related transactions:

1. `CONTRACT_DEPLOY`
2. `CONTRACT_INVOKE`
3. `CONTRACT_RESULT`
4. `COINBASE_CONTRACT_STATE_ROOT`

Contract call data is carried in OP_RETURN:

```text
OP_RETURN | SAT20_MAGIC_NUMBER | CONTENT_TYPE | CONTENT
```

Current content types:

```text
CONTENT_TYPE_CONTRACT_DEPLOY     = OP_DATA_31
CONTENT_TYPE_CONTRACT_INVOKE     = OP_DATA_32
CONTENT_TYPE_CONTRACT_RESULT     = OP_DATA_33
CONTENT_TYPE_CONTRACT_STATE_ROOT = OP_DATA_34
```

If a transaction has no contract OP_RETURN but contains outputs sent to valid contract addresses, those outputs may be interpreted as default invocations. A default invocation carries no explicit action or parameters; its business meaning is defined by the contract type and the contract instance.


CONTRACT_DEPLOY
----

`CONTRACT_DEPLOY` deploys a contract.

A deploy transaction must include contract type, contract content, version, deployer, random value, and gas limit. Each contract type may define its own deploy payload, but contract address generation and initial state initialization must be deterministic.

Every valid deployment must be recorded by a canonical `CONTRACT_RESULT` in the same block.


CONTRACT_INVOKE
----

`CONTRACT_INVOKE` calls a contract.

The call transaction must contain at least one output to the invoked contract address. This output is the gas/funding output and is used to:

1. Bind the Call TX and Result TX.
2. Carry assets transferred into the contract by this call.
3. Prepay the contract call fee.
4. Serve as an input to `CONTRACT_RESULT` when settlement is required.

The invoked contract address is not written in OP_RETURN. Nodes determine the invoked contract from the Call TX output sent to a contract address.

An explicit `CONTRACT_INVOKE` can call only one contract. If multiple outputs are sent to contract addresses, they must all belong to the same contract address. A default invocation has no explicit payload, so each contract output may independently trigger the default behavior of the corresponding contract.

Caller identity is resolved from the address of the previous output spent by the last input of the call transaction. The consensus path does not use witness public keys, signature public keys, or other replaceable fields as the caller/invoker source. If the previous output of the last input is unavailable or cannot be decoded to an address, the call is invalid.


CONTRACT_RESULT
----

`CONTRACT_RESULT` records VM execution results and settles asset transfers.

Rules:

1. `CONTRACT_RESULT` is constructed by the block producer. Externally submitted `CONTRACT_RESULT` transactions are not accepted by mempool.
2. `CONTRACT_RESULT` must appear after the `CONTRACT_DEPLOY` and `CONTRACT_INVOKE` transactions that it settles.
3. For ordinary deploys and invokes, `CONTRACT_RESULT` must be in the same block as the transactions that it settles.
4. When settling `CONTRACT_INVOKE`, the corresponding gas/funding output must be an input of the Result TX.
5. Result TX outputs must match the asset-transfer result derived by local replay.
6. Result TX may batch multiple execution items. Batch order must match protocol execution order.
7. A registered trigger may produce `CONTRACT_RESULT` after it becomes due even when the block contains no ordinary contract transaction. Validating nodes must identify the contract type and contract address from the contract UTXO spent by the Result TX, then replay the due trigger from previous state, current block height, and confirmation time. An isolated Result TX is invalid if the trigger cannot be proven to exist, be due, and still be valid.

The OP_RETURN of `CONTRACT_RESULT` contains only execution status summary and result count. Call ids, contract addresses, inputs, outputs, full trace, and asset-transfer details are derived from block transactions, local replay, and node indexer data.


State Root
----

The contract state root is written in coinbase. The block header is not changed.

A block may contain template, EVM, and Agent deploy/invoke/result transactions. Current execution order:

1. Execute template contract transactions and generate template Result TXs.
2. Execute EVM contract transactions and generate EVM Result TXs.
3. Execute Agent contract transactions and generate Agent Result TXs.
4. Combine the template state root, EVM state root, and Agent state root into one contract state root.
5. Write the final contract state root into coinbase.


Gas and Fees
----

Smart contracts use a unified gas asset.

Fee types:

1. Contract call fee.
2. VM execution fee.
3. Result TX packaging fee.
4. Trigger execution fee.

Rules:

1. Gas fees are paid by the caller or the contract itself.
2. Gas price is fixed by protocol in the first phase.
3. Gas fees are paid to the block producer.
4. The fee portion of the gas/funding output does not enter the contract asset balance.
5. A successful call with no asset transfer may omit Result TX.
6. Calls that produce asset transfers, refunds, reverts, or out of gas must generate Result TX.


Canonical Result TX
----

All nodes must construct and verify canonical Result TXs using the same rules.

Input selection:

1. Include the gas/funding output of the settled `CONTRACT_INVOKE` first.
2. Select only available UTXOs under the current contract address.
3. Group UTXOs by asset name.
4. Within the same asset, sort by confirmation height, txid, and vout.
5. Select from the sorted UTXO list until asset-transfer and fee requirements are met.
6. Stop selecting after requirements are met.

Output rules:

1. Output VM result asset transfers first.
2. If one execution item produces multiple outputs, use VM-defined order.
3. Change outputs return to the same contract address.
4. Change outputs are placed after asset-transfer outputs.
5. The OP_RETURN output is last.


Block Construction and Validation
----

Block producers must:

1. Select regular transactions.
2. Select contract-related transactions.
3. Execute contract deploys and invokes in block order.
4. Check registered triggers that are due in the current block, and actively execute due triggers even when the block has no other contract transaction.
5. Generate canonical Result TXs for execution items that require settlement.
6. Calculate the final contract state root.
7. Write the contract state root into coinbase and claim gas fees.

Validating nodes must:

1. Parse contract transactions in the block.
2. Replay contract execution in protocol order.
3. For Result TXs without same-block deploy/invoke transactions, determine the contract type from the spent contract UTXO and replay the corresponding due trigger.
4. Construct local canonical Result TXs.
5. Compare block Result TXs with locally derived Result TXs.
6. Calculate the local contract state root.
7. Check the state root commitment in coinbase.
8. Check gas fees claimed by coinbase.

Any mismatch makes the block invalid.


Testnet Integration Checklist
----

Before enabling contracts on testnet, node configuration must confirm at least:

1. Contract parsing, block validation, and block-producer Result Builders are enabled and use the same network parameters and contract address prefix.
2. Coinbase writes the unified contract state root, and validating nodes check the state-root commitment.
3. EVM trigger scanning runs for every candidate block, not only when the block contains ordinary contract transactions.
4. Agent contracts are configured with CoreNode address, CoreNode public key, Agent fee address, bootstrap fee address, and chain parameters.
5. Agent `confirm` carries CoreNode public key and signature, and nodes verify the public key, address, and signature.
6. Caller/invoker resolution depends on the previous-output address of the last input, and the node UTXO view must provide that previous-output script.
7. Mempool rejects externally submitted `CONTRACT_RESULT`. Result TXs are inserted only by the block-producing flow.
8. Local tests cover deploy, invoke, result-only trigger, Agent ready/confirm, EVM state root, Template state root, and mixed-block state root.
9. Gas parameters, block gas limit, contract Result packaging fee, and trigger packaging fee are fixed before launch.
10. Nodes, wallets, block explorers, asset explorers, and markets use the same contract transaction parsing and state-query fields.


Wallet, Explorer, and Market API Contract
----

Wallets, block explorers, asset explorers, and markets need at least the following shared interface semantics:

1. Contract deployment API: returns deploy txid, contract type, contract address, deployer, payload hash, initial state, and same-block Result status.
2. Contract invocation API: builds `CONTRACT_INVOKE`, includes the gas/funding output sent to the contract address, and displays the caller/invoker resolved from the previous output of the last input.
3. Contract result query API: queries canonical Result TX, status, asset transfers, gas fee, and error details by txid, contract address, call id, or trigger id.
4. Contract state query API: queries contract type, current state, state root, last update time, balance, and type-specific state by contract address.
5. EVM API: provides EVM-address and SatoshiNet-contract-address conversion, ABI calldata construction, event/log queries, EVM storage/code hash queries, and trigger queries.
6. Agent API: provides prediction deploy/ready/bet/confirm state, bet aggregation, candidate outcomes, confirmation evidence, CoreNode signature, and settlement/refund result queries.
7. Asset API: queries all UTXO asset balances under a contract address and distinguishes contract pool funds, gas/funding inputs, Result outputs, and change outputs.
8. Market API: markets only display contracts that are in `Ready` or another type-defined publicly displayable state. Before betting, buying, or transferring assets, markets must query latest node state and balance before building transactions.

Returned transaction, state, and asset results must be traceable to chain transactions, canonical Result TXs, and block state roots. They must not rely only on frontend-local inference.


Indexer Persistence and Query Requirements
----

Indexers must build a queryable on-chain view for contracts, including at least:

1. Contract deployment table: contract address, contract type, deploy txid, deployer, payload, payload hash, creation height, creation time, and initial Result status.
2. Contract invocation table: invoke txid, contract address, caller/invoker address, action, payload, gas/funding output, confirmation height, and execution status.
3. Contract Result table: result txid, contract address, result type, call id, trigger id, input UTXOs, output asset transfers, gas fee, execution status, error details, height, and time.
4. Contract state table: contract address, contract type, state root, current state, type-specific state, update time, and last Result txid.
5. EVM extension tables: EVM address mapping, code hash, storage root, logs/events, registered triggers, and trigger execution history.
6. Agent extension tables: Agent contract content hash, ready result, bet records, outcome aggregation, confirm records, CoreNode public key, CoreNode signature, result hash, and settlement/refund details.
7. Asset view: contract-address UTXO asset balance, locked pool funds, Result transfers, change, and fee collection.
8. Reorg handling: indexers must roll back contract deployments, invocations, Results, state, triggers, and asset views by block height.

Query APIs should support lookup by contract address, txid, caller address, contract type, state, height range, trigger id, Agent outcome, and EVM event topic.


Mempool Rules
----

Mempool must reject:

1. `CONTRACT_DEPLOY` or `CONTRACT_INVOKE` with invalid payload format.
2. Explicit `CONTRACT_INVOKE` without an output to a contract address.
3. Contract invocation sent to a non-existing contract address.
4. One explicit `CONTRACT_INVOKE` sent to multiple different contract addresses.
5. Missing gas limit or gas limit above protocol maximum.
6. Insufficient gas/funding.
7. Invalid gas asset type.
8. Calls that cannot pass local simulation.
9. Externally submitted `CONTRACT_RESULT`.
