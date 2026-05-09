EVM Smart Contracts
====

This document defines the first-phase protocol design for EVM-compatible smart contracts on SatoshiNet.

The goal of SatoshiNet EVM contracts is to provide an EVM-compatible execution environment on top of SatoshiNet's UTXO model. The EVM executes the contract state machine, UTXOs represent the source of truth for real assets and asset settlement, and transactions are the only external events that drive contract state changes.


Contract Type Boundary
----

EVM contracts and channel contracts are two different contract types on SatoshiNet.

Channel contracts are based on the RSMC/channel collaboration model. Their core purpose is to coordinate multi-party signatures, channel state, asset transcending, liquidity pools, AMM flows, and other protocol actions. Their asset-control and security boundaries come from channels, multisig, commitment transactions, and STP workflows.

EVM contracts are an independent virtual-machine contract type on SatoshiNet. They are compatible with EVM execution semantics, ABI, the Solidity development model, and the EVM contract ecosystem, but they do not copy Ethereum's account-based asset model. EVM contracts only execute state-machine logic; real assets are represented and settled by SatoshiNet's UTXO model.

This document discusses SatoshiNet EVM contracts. It is not an EVM version of channel contracts, nor does it embed EVM inside channel contracts.


Core Model
----

1. UTXO is the only source of on-chain assets.
2. An EVM contract is a state machine.
3. Contract calls are driven by transactions, while contract triggers are driven by the block environment.
4. All nodes must deterministically replay EVM execution.
5. EVM cannot directly spend UTXOs. It can only produce asset-transfer intents.
6. Asset-transfer intents must be settled at the UTXO layer through a canonical Result TX.
7. Contract UTXOs are not unlocked by traditional private-key signatures. They are authorized by EVM execution results.
8. The first phase uses the same-block Result TX settlement model.
9. In the first phase, the EVM state root is placed in the coinbase transaction as a block-level state commitment, without changing the block header structure.


Address Rules
----

EVM internal addresses must remain 20 bytes to remain compatible with Solidity, ABI, event logs, CREATE, and CREATE2.

1. `msg.sender` is a 20-byte EVM address.
2. `address(this)` is a 20-byte EVM contract address.
3. Indexed addresses in event topics are encoded according to the EVM ABI.
4. CREATE and CREATE2 generate 20-byte EVM contract addresses.

SatoshiNet external contract addresses use the following new contract address format:

```text
ca + version + type + hash
```

Testnet contract addresses use:

```text
tc + version + type + hash
```

Address fields:
1. The mainnet prefix is `ca`.
2. The testnet prefix is `tc`.
3. The current `version` is `1`.
4. `type` identifies the contract type. EVM contracts use a dedicated type.
5. `hash` is the 20-byte EVM address.

The internal 20-byte EVM address and the external SatoshiNet `ca/tc` address must support lossless conversion.

chain id, vm version, and code hash are not included in the address hash:
1. chain id is distinguished by the network and address prefix.
2. vm version is represented by the address version, type, and protocol upgrade rules.
3. code hash is committed by the EVM_DEPLOY transaction, contract state, and Result TX state commitment.


Asset Rules
----

Asset names are encoded as strings in ABI, using the result of SatoshiNet's asset-layer `AssetName.String()`.

The asset name for satoshi is fixed as:

```text
::
```

The first phase supports all SatoshiNet-recognized assets that can be represented by UTXOs, including:
1. satoshi, with asset name `::`.
2. ORDX assets.
3. Runes assets.
4. BRC20 assets.
5. Other assets recognized by the SatoshiNet asset layer and representable by UTXOs.

An EVM contract may maintain an application-level asset ledger internally, but the real asset balance must be based on the UTXO set under the contract address.


Transaction Types
----

EVM contract-related transactions are divided into four types:

1. `EVM_DEPLOY`
2. `EVM_INVOKE`
3. `EVM_RESULT`
4. `COINBASE_EVM_STATE_ROOT`


Contract Triggers
----

EVM contracts may define built-in triggers. A trigger does not depend on an external Call TX. It is triggered by block height, block time, or other protocol-allowed deterministic on-chain conditions.

Triggers express automatic execution logic inside a contract. For example, a vault contract can define a rule that automatically releases assets to a specified address at a certain block height.

Trigger rules:
1. Trigger conditions must be fully determined by deterministic on-chain data.
2. Triggers must not depend on local node time, external services, randomness, or any other non-deterministic input.
3. Trigger execution must be deterministically replayed by all nodes.
4. Trigger execution may produce Asset Intents.
5. If a trigger produces Asset Intents, they must be settled by an `EVM_RESULT` in the same block.
6. A trigger does not need an `EVM_INVOKE`, so it has no Call TX gas/funding input.
7. Trigger execution fees and Result TX packaging fees are deducted from the contract's own gas-asset UTXOs.
8. If the contract does not have enough gas assets to pay the trigger execution fee and Result TX packaging fee, the trigger must not execute.
9. If the same contract has both triggers and `EVM_INVOKE` transactions in the same block, `EVM_INVOKE` transactions are executed first in block transaction order, then triggers due at that block height are executed.


EVM_DEPLOY
----

`EVM_DEPLOY` deploys an EVM contract.

The OP_RETURN of `EVM_DEPLOY` must carry:
1. init code.
2. constructor parameters.
3. deployer EVM address.
4. gas limit.
5. deployment nonce.

After successful execution, `EVM_DEPLOY` must produce:
1. a 20-byte EVM contract address.
2. a SatoshiNet `ca/tc` contract address.
3. code hash.
4. initial storage root.

Every `EVM_DEPLOY` must use `EVM_RESULT` to record the deployment result, contract address, and state root.


EVM_INVOKE
----

`EVM_INVOKE` invokes an EVM contract.

The OP_RETURN of `EVM_INVOKE` must carry:
1. calldata.
2. gas limit.
3. call nonce.

`EVM_INVOKE` must have at least one output to the invoked contract address as the gas/funding output. This output has three purposes:
1. It provides the UTXO-level binding between the Call TX and Result TX.
2. It carries the assets transferred to the contract by this call.
3. It prepays the contract call fee and Result TX packaging fee.

The invoked contract address is not written into the OP_RETURN of `EVM_INVOKE`. Nodes must determine the invoked contract from the Call TX output to the contract address.

Each `EVM_INVOKE` can invoke only one EVM contract. If an `EVM_INVOKE` contains multiple outputs to EVM contract addresses, all such outputs must belong to the same contract address; otherwise the transaction is invalid.

At the protocol layer, the gas/funding output is split into a fee portion and a funding portion. The fee portion belongs to the block producer and is claimed through coinbase. Only the remaining portion after fee deduction enters the contract UTXO set.

If `EVM_INVOKE` produces an asset transfer, or execution fails and requires a refund, it must be settled by an `EVM_RESULT` in the same block.

If `EVM_INVOKE` executes successfully and produces no asset transfer, no `EVM_RESULT` is generated. The call only needs to pay the contract call fee and does not need to pay the Result TX packaging fee. After deducting the contract call fee, the remaining assets in the gas/funding output directly become contract UTXOs and may be used by the contract later.


EVM_RESULT
----

`EVM_RESULT` records EVM execution results and settles asset-transfer intents.

`EVM_RESULT` may batch-settle multiple `EVM_DEPLOY`, `EVM_INVOKE`, and contract trigger execution items. Execution items settled by the same `EVM_RESULT` must be recorded in protocol execution order.

`EVM_RESULT` must appear after all `EVM_DEPLOY` and `EVM_INVOKE` transactions it settles, and must be in the same block as the transactions it settles.

If an `EVM_INVOKE` is settled by an `EVM_RESULT`, the gas/funding output that the `EVM_INVOKE` sent to the contract address must be an input of that `EVM_RESULT`.

If an `EVM_RESULT` batch-settles multiple `EVM_INVOKE` transactions, the gas/funding outputs of all those `EVM_INVOKE` transactions must be inputs of that `EVM_RESULT`.

If `EVM_RESULT` settles a contract trigger, no Call TX gas/funding input is required. Nodes must verify that the trigger is executable in the current block through contract state, trigger conditions, block environment, and local EVM replay.

The OP_RETURN of `EVM_RESULT` only commits the minimal result data. Information that can be derived from the transaction itself, Result TX inputs and outputs, or local EVM replay is not written into OP_RETURN.

The OP_RETURN of `EVM_RESULT` must contain:
1. execution status summary.
2. batch result count.
3. optional error code or revert reason digest.

call id rules:
1. The call id of `EVM_INVOKE` is generated from invoke txid, contract output vout/index, and the contract address corresponding to that output.
2. The call id of `EVM_DEPLOY` is generated from deploy txid and contract address.
3. The call id of a contract trigger is generated from contract address, trigger id, and trigger block height.
4. `EVM_RESULT` does not write the call id list into OP_RETURN.
5. Nodes must determine which `EVM_INVOKE` transactions are settled by looking backward from the gas/funding outputs in the inputs of `EVM_RESULT`.
6. The `EVM_RESULT` corresponding to `EVM_DEPLOY` must be determined by transaction order and result count within the same block.
7. The `EVM_RESULT` corresponding to a contract trigger must be determined by contract state, trigger condition, and trigger block height.
8. Batch result order must match protocol execution order.

The following information is not written into the OP_RETURN of `EVM_RESULT`:
1. call id list.
2. contract address list.
3. gas used.
4. pre-execution state root.
5. post-execution state root.
6. code hash or contract root.
7. intent commitment.
8. asset settlement commitment.
9. logs commitment.

Nodes must obtain this information from:
1. `EVM_RESULT` inputs.
2. `EVM_RESULT` outputs.
3. the `EVM_DEPLOY` and `EVM_INVOKE` transactions settled in the same block.
4. local deterministic EVM replay.
5. the node's internal indexer database.

The full trace is not written on-chain. Nodes must obtain the full execution details by local replay.


COINBASE_EVM_STATE_ROOT
----

In the first phase, SatoshiNet does not change the block header structure. The EVM state root must be placed in the coinbase transaction as a block-level state commitment.

The EVM state root in coinbase represents the final EVM state after all EVM contract transactions in the block have been executed and settled.

When validating a block, nodes must independently replay all EVM contract transactions in the block and check that the locally calculated EVM state root matches the commitment in coinbase.

This design does not change the current 80-byte block header structure, does not change block hash calculation, and does not break existing block header parsing logic. External systems such as block explorers only need to recognize the EVM state root commitment in coinbase.


Call Data
----

EVM contract call data is carried through OP_RETURN.

Data format:

```text
OP_RETURN | SAT20_MAGIC_NUMBER | CT_TYPE | CONTENT
```

Field definitions:

```text
SAT20_MAGIC_NUMBER      = OP_16
CONTENT_TYPE_EVM_DEPLOY = OP_DATA_31
CONTENT_TYPE_EVM_INVOKE = OP_DATA_32
CONTENT_TYPE_EVM_RESULT = OP_DATA_33
CONTENT                 = contract content / call parameters / call result
```

Call parameters use standard ABI encoding:

```text
4 bytes methodID + ABI encoded params
```


Contract UTXO Unlocking Rules
----

UTXOs under a contract address are not unlocked by traditional private-key signatures. When spending contract UTXOs, nodes must verify that the spend is authorized by an EVM execution result.

Spending a contract UTXO must satisfy:
1. The current block contains the corresponding `EVM_DEPLOY` or `EVM_INVOKE`, or contains a contract trigger executable at the current block height.
2. The corresponding transaction calls or deploys the contract, or the corresponding trigger belongs to the contract.
3. If the authorization source is `EVM_INVOKE`, `EVM_RESULT` must spend the gas/funding output sent by the corresponding Call TX to the contract address.
4. If the authorization source is a contract trigger, `EVM_RESULT` does not need to spend a Call TX gas/funding output, but it must spend the contract's own gas-asset UTXO to pay the trigger execution fee and Result TX packaging fee.
5. All nodes replay EVM and obtain the same asset-transfer intents.
6. `EVM_RESULT` only spends contract UTXOs authorized by EVM.
7. `EVM_RESULT` outputs exactly match the EVM execution result.
8. `EVM_RESULT` carries correct EVM_RESULT data.

A contract address is essentially a UTXO set controlled by an EVM state machine.


Asset Intent
----

EVM cannot directly spend UTXOs. It can only generate Asset Intents.

An Asset Intent contains:
1. call id.
2. intent index.
3. from contract.
4. to address.
5. asset name.
6. amount.
7. extra data.

After replaying EVM, nodes must generate the canonical Result TX from Asset Intents.

All low-level asset interfaces are implemented through precompiled contracts. Precompiled contracts are responsible for:
1. querying contract UTXO asset state.
2. generating asset-transfer intents.
3. participating in deterministic Result TX verification.

Precompiled interfaces include:
1. `assetBalance(contract, assetName)`.
2. `transferAsset(to, assetName, amount)`.
3. `receivedAsset(assetName)`.


Canonical Result TX Rules
----

All nodes must construct canonical Result TXs using the same rules.

Input selection rules:
1. The gas/funding output sent by the settled `EVM_INVOKE` to the contract address must be included first.
2. Only available UTXOs under the current contract address can be selected.
3. UTXOs are grouped by asset name.
4. Within the same asset, UTXOs are sorted by confirmation height, txid, and vout.
5. UTXOs are selected from the head of the sorted list until the accumulated amount is sufficient for the intents and fees.
6. Once the accumulated amount is sufficient, no more UTXOs may be selected.

If `EVM_RESULT` settles a contract trigger, there is no `EVM_INVOKE` gas/funding output. In this case, input selection starts from available UTXOs under the contract address, and the selected inputs must include enough gas-asset UTXOs to pay the trigger execution fee and Result TX packaging fee.

Output ordering rules:
1. Outputs for assets specified by intents come first.
2. If the same call produces multiple intents, they are ordered by intent index.
3. Change outputs must return to the same contract address.
4. Change outputs appear after asset outputs.
5. The OP_RETURN output is always the last output.

Change rules:
1. If the total input amount of contract UTXOs is greater than the total intent amount, the remaining assets must return to the same contract address.
2. Change cannot be freely specified by the block producer.
3. Dust rules follow existing SatoshiNet transaction rules.

Fee rules:
1. Contract call fees are prepaid by the caller.
2. Result TX packaging fees are prepaid by the caller.
3. The caller must provide enough gas/funding when invoking the contract.
4. gas/funding must cover the contract call fee.
5. If the call requires a Result TX, gas/funding must also cover the Result TX packaging fee.
6. If gas/funding is insufficient, the `EVM_INVOKE` is invalid and must not enter a block.


EVM Executor and StateDB
----

The first phase uses go-ethereum's `core/vm` as the EVM executor.

Implementation requirements:
1. Do not change `core/vm` EVM semantics.
2. Implement a SatoshiNet custom StateDB.
3. StateDB must maintain contract state, contract logs, and state commitments.
4. Contract assets are represented by UTXO sets, not by EVM account balances.

EVM balance exists as a compatibility view, but it must not bypass the SatoshiNet UTXO asset ledger.

`msg.value` rules:
1. `msg.value` is supported.
2. `msg.value` represents the satoshi amount transferred by the current Call TX to the contract address.
3. The asset name for satoshi is `::`.

`address.balance` rules:
1. `address.balance` returns the satoshi balance.
2. `address(this).balance` represents the current contract address's spendable satoshi total.
3. This value is calculated from UTXOs under the contract address.

Multi-asset balance rules:
1. Multi-asset balances must be queried through precompiled contracts.
2. A contract may maintain multi-asset information in storage.
3. The real asset balance is based on the result of the precompiled interface counting the contract UTXO set.
4. Multi-asset records in contract storage must match the result of the precompiled interface counting all UTXOs.
5. If they differ, Result TX validation uses the precompiled interface and UTXO asset state as the source of truth.

ERC20 balance rules:
1. ERC20 balances inside Solidity contracts are only contract storage.
2. ERC20 balances may be used as an application-level ledger.
3. ERC20 balances are not SatoshiNet native asset balances.
4. If an ERC20 transfer needs to be redeemed as a real SatoshiNet asset transfer, it must generate an Asset Intent through a precompiled contract and be settled by a Result TX.


Gas and Fees
----

EVM must retain the gas mechanism to prevent complex-computation attacks.

Gas rules:
1. EVM opcode gas costs follow geth.
2. Every `EVM_DEPLOY` and `EVM_INVOKE` must declare a gas limit.
3. If node execution exceeds the gas limit, the execution result is out of gas.
4. Gas fees are paid by the caller.
5. Each block must have a total EVM gas limit.
6. The first phase uses a protocol-fixed gas price and does not use a dynamic fee market.
7. Gas fees are paid using a new dedicated asset.
8. The gas asset name is pending and will be written into the protocol using the result of `AssetName.String()`.
9. Gas fees belong to the block producer.

gas/funding fees are split into two classes:
1. contract call fee.
2. Result TX packaging fee.

The contract call fee pays for EVM execution cost. Every `EVM_DEPLOY` and `EVM_INVOKE` must pay the contract call fee. This fee is paid in the dedicated gas asset and belongs to the block producer.

The Result TX packaging fee pays for the block packaging cost of `EVM_RESULT`. Only calls that require `EVM_RESULT` settlement need to pay the Result TX packaging fee. This fee is paid in the dedicated gas asset and belongs to the block producer.

The fee portion of a gas/funding output does not enter the contract asset balance. When validating the contract UTXO set, nodes must first deduct the gas fees payable to the block producer. Only the remaining portion after deduction becomes contract UTXO assets.

Contract-related transactions are special transactions and do not additionally pay ordinary network fees in satoshi.

Block ordering rules:
1. Ordinary transactions are packaged first.
2. Contract-related transactions are placed after ordinary transactions.
3. Contract-related transactions are not sorted together with ordinary satoshi-fee transactions.
4. Contract-related transactions are sorted only within the contract transaction set according to gas rules.

When invoking a contract, the caller must provide enough gas assets as gas/funding. As long as the call transaction provides enough gas/funding, miners may accept the call and the corresponding Result TX.

An `EVM_INVOKE` that executes successfully and produces no asset transfer does not generate a Result TX, so it does not need to pay the Result TX packaging fee. After deducting the contract call fee, the remaining assets in the gas/funding output directly become contract UTXOs and may be used by the contract later.


Revert and Failure Handling
----

1. If EVM execution reverts, contract storage is not updated.
2. revert is also an EVM_RESULT and must be expressed through a Result TX.
3. A revert Result TX must refund assets other than gas.
4. Gas is not refunded on revert.
5. out of gas is handled as revert.
6. Gas fees must be deducted on out of gas.
7. Invalid calls are rejected at the mempool stage and must not enter a block.
8. If a block contains an invalid call, the block is invalid.
9. If a Result TX is missing or its refund outputs are incorrect, the entire block is invalid.


Mempool Rules
----

mempool must reject the following transactions:
1. `EVM_DEPLOY` or `EVM_INVOKE` with invalid calldata format.
2. `EVM_INVOKE` with no output to an EVM contract address.
3. `EVM_INVOKE` with output to a non-existent contract address.
4. A transaction where the same `EVM_INVOKE` outputs to multiple different EVM contract addresses.
5. A transaction with missing gas limit or gas limit above the protocol limit.
6. A transaction with insufficient gas/funding.
7. A transaction whose gas asset type does not match the protocol requirement.
8. A transaction that cannot pass local simulation.

mempool stores valid `EVM_DEPLOY` and `EVM_INVOKE` transactions, while `EVM_RESULT` is generated or selected by the block producer during block construction and deterministically verified by all nodes.


Block Construction Rules
----

When constructing a block, the block producer must:
1. select ordinary transactions first.
2. then select contract-related transactions.
3. sort contract-related transactions according to gas rules.
4. execute `EVM_DEPLOY` and `EVM_INVOKE` in block order.
5. generate `EVM_RESULT` for all `EVM_DEPLOY` transactions.
6. generate `EVM_RESULT` for `EVM_INVOKE` transactions that produce asset transfers, revert, or run out of gas.
7. not generate `EVM_RESULT` for `EVM_INVOKE` transactions that execute successfully and produce no asset transfer.
8. check contract triggers executable at the current block height.
9. generate `EVM_RESULT` for contract triggers that produce asset transfers.
10. construct canonical Result TXs according to deterministic rules.
11. write the final EVM state root into coinbase.
12. claim contract call fees, trigger execution fees, and Result TX packaging fees in coinbase.


Block Validation Rules
----

When validating a block, nodes must execute the following process:

1. Scan transactions in block order.
2. When `EVM_DEPLOY` is found, execute deployment logic and generate the contract address and initial state.
3. When `EVM_INVOKE` is found, parse the invoked contract address from the Call TX output and execute the EVM call.
4. After execution, obtain state diff, logs, gas used, and Asset Intents.
5. If the transaction requires `EVM_RESULT`, add it to the pending settlement queue.
6. Check executable contract triggers according to the current block height and contract state.
7. If a trigger requires `EVM_RESULT`, add it to the pending settlement queue.
8. When `EVM_RESULT` is found, determine the settled `EVM_INVOKE` transactions by looking backward from its gas/funding inputs.
9. Determine the settled `EVM_DEPLOY` transactions and contract triggers from the batch result count, block transaction order, and pending settlement queue.
10. Construct the canonical Result TX according to protocol rules.
11. Compare the Result TX in the block with the locally derived result.
12. If they differ, the block is invalid.
13. At the end of block scanning, if there are still unsettled `EVM_DEPLOY`, `EVM_INVOKE` transactions that require Result TX, or contract triggers that require Result TX, the block is invalid.
14. Calculate the final EVM state root.
15. Check whether the final EVM state root equals the commitment in coinbase.
16. If they differ, the block is invalid.
17. Calculate all contract call fees, trigger execution fees, and Result TX packaging fees payable by contract transactions and triggers in this block.
18. Check whether the amount of dedicated gas assets claimed in coinbase equals the total payable fee.
19. If they differ, the block is invalid.

Multiple contract calls in a block are executed serially in transaction order. A later `EVM_INVOKE` may read contract state already committed by earlier EVM transactions, but cannot read state from later transactions in the block.

Multiple calls to the same contract in the same block must execute serially.


Open Issues
----

The following details have not been finalized:

1. The name of the dedicated gas asset.
2. The issuance rules for the dedicated gas asset.
3. The concrete value of the fixed gas price.
4. Launch and test schedule.
