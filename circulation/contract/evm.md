EVM Contracts
====

EVM contracts are one SatoshiNet smart-contract execution type. The EVM executes Solidity/EVM state machines. The SatoshiNet UTXO model provides real asset sources and settlement.

EVM contracts follow the common [Smart Contracts](readme.md) protocol. This document defines EVM-specific rules only.


Contract Type
----

EVM contract type:

```text
ContractTypeEVM
```

EVM contracts are compatible with EVM execution semantics, ABI, Solidity development model, and EVM event model, but they do not use Ethereum's account-based asset model. Internal EVM state may maintain application-level ledgers, but real asset balances are based on UTXOs under the contract address.


Address Rules
----

EVM internal addresses remain 20 bytes for Solidity and ABI compatibility.

1. `msg.sender` is a 20-byte EVM address.
2. `address(this)` is a 20-byte EVM contract address.
3. Indexed addresses in event topics use EVM ABI encoding.
4. CREATE and CREATE2 generate 20-byte EVM contract addresses.

SatoshiNet external contract address format:

```text
ca/tc + version + ContractTypeEVM + hash
```

The EVM contract address hash is the 20-byte EVM address. The internal EVM address and external SatoshiNet `ca/tc` address must support lossless conversion.

chain id, vm version, and code hash are not included in the address hash:

1. chain id is distinguished by network and address prefix.
2. vm version is represented by address version, type, and protocol upgrade rules.
3. code hash is committed by deploy transaction, contract state, and state root.


Deployment
----

`CONTRACT_DEPLOY` deploys an EVM contract.

EVM deploy payload contains:

1. init code.
2. constructor parameters.
3. deployer EVM address.
4. gas limit.
5. deployment nonce.

Successful deployment produces:

1. 20-byte EVM contract address.
2. SatoshiNet `ca/tc` contract address.
3. code hash.
4. initial storage root.

Deployment result must be recorded by canonical `CONTRACT_RESULT` in the same block.


Invocation
----

`CONTRACT_INVOKE` calls an EVM contract.

EVM invoke payload contains:

1. calldata.
2. gas limit.
3. call nonce.

The call transaction must output to the invoked contract address as gas/funding output. The invoked contract address is parsed from the Call TX output and is not written into OP_RETURN.

EVM `msg.sender` is determined from the previous-output address spent by the last input of the call transaction, then mapped to a 20-byte EVM address. Nodes must not derive `msg.sender` from witness public keys.

For EVM contracts, a default invocation means an empty-calldata call to the contract address. Satoshis in the output become `msg.value`; other assets remain represented by the SatoshiNet asset layer and the precompiled asset interfaces.


StateDB and Executor
----

The first-phase EVM executor is based on go-ethereum `core/vm`.

Requirements:

1. Do not change EVM opcode semantics.
2. Implement SatoshiNet custom StateDB.
3. StateDB maintains storage, code, logs, and state root.
4. Contract assets are represented by UTXO sets, not by EVM account balance.


Assets
----

Asset names are ABI-encoded as strings using SatoshiNet asset-layer `AssetName.String()`.

The satoshi asset name is:

```text
::
```

The first phase supports assets recognized by SatoshiNet and representable by UTXOs:

1. satoshi.
2. ORDX assets.
3. Runes assets.
4. BRC20 assets.
5. Other assets recognized by the SatoshiNet asset layer.

EVM contracts may maintain ERC20 or other application-level balances internally. Those balances are not SatoshiNet native asset balances. To settle them as SatoshiNet assets, the contract must generate Asset Intents through asset interfaces and settle them through `CONTRACT_RESULT`.


Precompiled Asset Interfaces
----

EVM cannot directly spend UTXOs. It can only generate Asset Intents. Low-level asset access is provided through precompiled contracts.

Precompiled contracts:

1. Query contract UTXO asset state.
2. Generate asset-transfer intents.
3. Participate in deterministic Result TX verification.

First-phase interfaces:

1. `assetBalance(contract, assetName)`
2. `transferAsset(to, assetName, amount)`
3. `receivedAsset(assetName)`


`msg.value` and Balance
----

`msg.value` is the satoshi amount transferred to the contract address by the Call TX.

`address.balance` returns satoshi balance. `address(this).balance` is the total spendable satoshi amount under the current contract address, calculated from contract-address UTXOs.

Multi-asset balances must be queried through precompiled contracts. If asset records in storage differ from UTXO-derived balances, Result TX verification uses UTXO asset state and precompiled interfaces as the source of truth.


Gas
----

EVM keeps the gas mechanism.

Rules:

1. Opcode gas costs follow geth.
2. Every deploy and invoke declares gas limit.
3. Execution above gas limit returns out of gas.
4. Gas fees are paid by caller.
5. A block has an EVM gas limit.
6. Gas price is fixed by protocol in the first phase.
7. Gas fees use the unified gas asset.
8. Gas fees are paid to the block producer.


Triggers
----

EVM contracts may register height triggers through the protocol precompile. Triggers are stored in EVM state and participate in the EVM state root.

A height trigger contains at least:

1. trigger id.
2. target contract address.
3. trigger height.
4. trigger execution gas limit.
5. calldata invoked when the trigger fires.

Block producers must check due triggers in EVM state for every block they construct. Even if the candidate block contains no ordinary EVM deploy/invoke transaction, a due trigger must be executed and must produce a canonical `CONTRACT_RESULT` when settlement is required.

When validating an EVM Result TX without same-block EVM deploy/invoke transactions, nodes must determine the contract address from the EVM contract UTXO spent by the Result TX and replay the trigger from previous EVM state, current block height, and block confirmation time. The block is invalid if the trigger does not exist, is not due, has already been removed, has invalid gas limit, or replays to a different result.


Result TX and State Root
----

EVM contracts do not accept externally submitted `CONTRACT_RESULT` transactions in mempool.

The block producer executes EVM and constructs canonical `CONTRACT_RESULT` from Asset Intents. Validating nodes independently replay EVM and compare Result TX and state root.

The EVM state root, template state root, and Agent state root are combined into one contract state root and written into coinbase.
