# Smart Contract Protocol

This document defines the common SatoshiNet smart-contract protocol. Smart contracts are different from [channel contracts](../channel-contracts/): channel contracts manage public asset pools and coordinate user-initiated L1/L2 cross-layer actions, while smart contracts run in SatoshiNet's global execution environment.

Contract-type details:

1. [Template Contracts](template.md)
2. [EVM Contracts](evm.md)
3. [Natural Language Contracts](agent.md)

## Protocol Boundary

A SatoshiNet smart contract is a programmable consensus state machine on SatoshiNet. Contract state is maintained by SatoshiNet nodes, while asset inputs and settlement are represented by the SatoshiNet UTXO model.

Channel contracts coordinate public asset pools and L1/L2 actions. Smart contracts use on-chain transactions, VM state, canonical `CONTRACT_RESULT`, and block-level state roots. Asset transfers initiated by smart contracts are not unlocked by private-key signatures. They must be authorized by VM execution results.

## Contract Types

The first stage contains three contract families:

| Type                      | Execution model                                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Template Contract         | Native Go runtime embedded in SatoshiNet nodes                                                                                       |
| EVM Contract              | EVM executor                                                                                                                         |
| Natural Language Contract | AI Agent-assisted protocol contract. The first stage focuses on prediction-style Agent contracts using a single Core Node Agent path |

## Core Model

1. UTXOs are the source of on-chain assets.
2. The VM is the contract state machine.
3. `CONTRACT_DEPLOY` creates a contract.
4. `CONTRACT_INVOKE` calls a contract.
5. `CONTRACT_RESULT` settles asset transfers produced by VM execution.
6. `COINBASE_CONTRACT_STATE_ROOT` commits the post-execution contract state root in the coinbase transaction.
7. All nodes must deterministically replay contract execution.
8. Contract UTXOs are not spent by traditional private-key signatures; they are spent by VM-authorized results.

## Address Rules

Mainnet contract address format:

```
ca + version + type + hash
```

Testnet contract address format:

```
tc + version + type + hash
```

`ca` and `tc` are network prefixes, `version` is currently `1`, `type` is the contract type, and `hash` is defined by each contract family.

## Transaction Types

Contract transactions use OP\_RETURN envelopes:

```
OP_RETURN | SAT20_MAGIC_NUMBER | CONTENT_TYPE | CONTENT
```

The contract transaction families are:

1. `CONTRACT_DEPLOY`
2. `CONTRACT_INVOKE`
3. `CONTRACT_RESULT`
4. `COINBASE_CONTRACT_STATE_ROOT`

`CONTRACT_INVOKE` OP\_RETURN should carry only action, nonce, gas limit, and non-economic parameters that cannot be derived from outputs. Asset name, asset amount, sat amount, and gas/funding outputs come from the Call TX outputs to the contract address and should not be duplicated in OP\_RETURN.

If a transaction has no contract OP\_RETURN but contains an output to a valid contract address, that output can be interpreted as a default invocation. Its business meaning is defined by the specific contract type and instance.

Deploy transactions may split large contract content across multiple contract OP_RETURN outputs. Ordinary transactions and contract invocations should not abuse OP_RETURN outputs. Except for protocol-defined deployment chunking, the mempool policy limits ordinary paths; the current ordinary path should not exceed four OP_RETURN outputs.

## Contract Close and Profit Distribution

Smart contracts use a common close rule. A `close` invocation can be made only by the contract deployer. Before closing, each contract type may return assets with clear ownership according to its own state rules, such as open orders, LP shares, or other user-owned positions.

After close processing, remaining contract-managed assets without clear user ownership are treated as contract profit and distributed 60% to the deployer and 40% to the bootstrap recipient. Assets held at the contract address that exceed the runtime-managed asset records are not used for contract business settlement; on close they are sent to the bootstrap recipient for later handling.

This rule applies to Template, EVM, and Natural Language contracts. A contract family may define which assets have clear user ownership before close, but it cannot change the common handling of unowned profit and unmanaged assets.

## `CONTRACT_DEPLOY`

`CONTRACT_DEPLOY` deploys a contract. The deployment payload contains the contract type, content, version, deployer, nonce, and gas limit. Each contract family defines its own deployment payload, but the contract address and initial state must be deterministically derived.

Every valid deployment must be recorded by a canonical `CONTRACT_RESULT` in the same block.

## `CONTRACT_INVOKE`

`CONTRACT_INVOKE` calls a contract. The call transaction must include exactly one gas/funding output to the called contract address. That output binds the Call TX to the Result TX, carries assets sent to the contract, prepays the contract call fee, and can be spent by `CONTRACT_RESULT`.

The called contract address is determined from outputs, not from OP\_RETURN. An explicit `CONTRACT_INVOKE` can call only one contract, and the called contract must have exactly one gas/funding output in that call. This keeps the economic input source, caller, refund recipient, and Result TX binding deterministic. Default invocations may trigger per-output behavior.

Caller identity is derived from the address of the previous output spent by the last input of the call transaction. Consensus does not use witness public keys or replaceable signature fields as caller identity.

## `CONTRACT_RESULT`

`CONTRACT_RESULT` records VM execution results and settles asset transfers.

Rules:

1. Block producers construct `CONTRACT_RESULT`; externally submitted result transactions do not enter the mempool.
2. `CONTRACT_RESULT` appears after the deploy or invoke transaction it settles.
3. Ordinary deploy and invoke result transactions must be in the same block as the settled transaction.
4. For `CONTRACT_INVOKE`, the gas/funding output sent to the contract address is spent by the Result TX.
5. Result outputs must match the locally replayed asset-transfer result.
6. Result TX can batch multiple execution items, and the batch order must match the protocol execution order.
7. Expired triggers may produce `CONTRACT_RESULT` without ordinary contract calls in the same block, but validators must be able to prove the trigger exists, is due, and remains valid.

The Result TX OP\_RETURN stores execution summary and result count. Call id, contract address, inputs, outputs, trace, and asset-transfer details are derived from block transactions, local replay, and node indexers.

## State Root

Contract state roots are written into the coinbase transaction without changing the block-header structure.

Within a block, template contracts, EVM contracts, and Agent contracts can all execute. The current order is:

1. Execute template-contract transactions and generate Result TXs.
2. Execute EVM-contract transactions and generate Result TXs.
3. Execute Agent-contract transactions and generate Result TXs.
4. Combine template, EVM, and Agent state roots into a unified contract state root.
5. Commit the final state root in coinbase.

## GAS and Fees

Smart contracts use a unified GAS asset for fees.

Fee categories include contract call fee, VM execution fee, Result TX packaging fee, and trigger execution fee. Fees are paid by the caller or by the contract itself, depending on the contract rule. The first stage uses a protocol-defined fixed gas price. Fees are paid to block producers. Fee portions inside a gas/funding output do not enter the contract asset balance.

Default limits are: per-deploy/invoke gas limit `50,000,000`, per-trigger execution gas limit `5,000,000`, and per-block EVM execution gas limit `1,000,000,000`.

If execution succeeds and no asset transfer is needed, a Result TX may be unnecessary. If asset transfer, refund, revert, or out-of-gas settlement is needed, a Result TX must be generated.

## Canonical Result TX

All nodes must construct and verify canonical Result TXs using the same rules.

Input selection prioritizes the gas/funding output sent to the contract address, then selects available UTXOs at the same contract address grouped by asset name and ordered by confirmation height, txid, and vout.

Output rules:

1. VM-specified asset transfers are output first.
2. Multiple outputs from the same execution item follow VM-defined order.
3. Change returns to the same contract address.
4. Change outputs are placed after asset-transfer outputs.

Any block whose Result TXs cannot be reproduced by validators is invalid.

## Mempool Policy

The mempool performs structural, address, fee, and basic protocol checks. Business-level invalid contract actions should not block the mempool or block production. Execution should treat them as no-op calls or produce deterministic refund results according to the contract rule. `CONTRACT_RESULT` is different: Result TXs are produced by nodes, and a mismatch makes the block invalid.

The mempool must reject:

1. Malformed `CONTRACT_DEPLOY` or `CONTRACT_INVOKE` payloads.
2. Explicit invocations without exactly one output to the called contract address.
3. Calls to non-existent contract addresses.
4. Missing or out-of-range gas limits.
5. Insufficient gas/funding.
6. Invalid gas asset type.
7. Excessive OP_RETURN outputs outside deployment chunking.
8. Externally submitted `CONTRACT_RESULT` transactions.

## Wallet, Explorer, and Market Interfaces

Wallets, explorers, asset browsers, and markets should share these interface semantics:

1. Deployment APIs return deploy txid, contract type, contract address, deployer, payload hash, initial state, and same-block Result status.
2. Invocation APIs build `CONTRACT_INVOKE` transactions with the single gas/funding output to the contract address and display the caller derived from the last input's previous output.
3. Result queries expose canonical Result TXs, state changes, asset transfers, gas fees, and execution errors.
4. State queries return contract type, current state, state root, update height, balances, and type-specific state view.
5. EVM interfaces provide SatoshiNet contract address mapping, ABI calldata construction, logs/events, storage/code hash, trigger queries, compiler config, and source metadata where available.
6. Agent interfaces expose prediction deploy/ready/bet/confirm state, aggregated bets, outcomes, confirmation material, Core Node caller identity, and settlement/refund results.
7. Asset interfaces query all UTXO-backed balances at a contract address and distinguish contract-managed assets, gas/funding inputs, Result outputs, and change.
8. Markets should display only contracts whose runtime exists and whose type-specific status is publicly displayable; before value-moving actions they must query the latest node state and balances.

These interfaces must be traceable to on-chain transactions, canonical Result TXs, and block state roots rather than relying only on frontend inference.
