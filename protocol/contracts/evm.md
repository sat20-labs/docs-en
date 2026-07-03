# EVM Contracts

EVM Contracts bring an Ethereum-compatible execution environment into SatoshiNet while keeping SatoshiNet's UTXO asset settlement model.

EVM Contracts follow the common [Smart Contract Protocol](./). This page defines the EVM-specific boundary.

## Design Goal

SatoshiNet EVM support is intended to give developers a familiar smart-contract environment while preserving Bitcoin-native asset entry and SatoshiNet consensus settlement.

The EVM executor handles bytecode execution and EVM state. SatoshiNet consensus still decides which contract calls are valid, how GAS is paid, and how asset-transfer Result TXs are constructed.

## Contract Type

EVM Contracts are deployed with an EVM contract type. The contract address uses the common SatoshiNet contract address format:

```
ca/tc + version + evm-contract-type + hash
```

The address hash is derived from EVM deployment content, deployer, and deployment nonce. The address is a SatoshiNet contract address, not a direct Ethereum account address.

## Deployment

`CONTRACT_DEPLOY` for EVM includes EVM bytecode or creation code, deployer, deployment nonce, gas limit, and any required initialization payload.

Nodes execute deployment through the EVM executor, initialize EVM state, and record the result with canonical `CONTRACT_RESULT` and state root.

## Invocation

`CONTRACT_INVOKE` calls an EVM Contract. The Call TX output to the contract address binds the invocation, provides GAS/funding, and can carry SatoshiNet assets into the contract.

EVM calldata, method selector, deadline, slippage, or other non-economic parameters can be carried in the invoke payload. Economic parameters are derived from transaction outputs to the contract address.

## Asset Boundary

The EVM account model and the SatoshiNet UTXO model must stay consistent:

1. EVM execution decides logical contract state.
2. SatoshiNet UTXOs represent actual asset inputs and outputs.
3. Asset transfers caused by EVM execution are settled by canonical `CONTRACT_RESULT`.
4. Validators replay EVM execution and reject blocks whose Result TX does not match replayed output.

This avoids treating off-chain or non-canonical EVM traces as final asset transfers.

## Native Asset Precompile

EVM contracts cannot spend UTXOs directly. They can only query contract asset state and emit asset-transfer intents through the SatoshiNet native asset precompile.

The first-stage asset precompile interface is:

1. `balanceOf(address, assetName)`: returns the available balance of the specified asset for the contract address derived from the EVM address.
2. `transferAsset(assetName, to, amount, extraData)`: declares an asset-transfer intent. This call does not directly spend UTXOs. The final asset movement must be settled by canonical `CONTRACT_RESULT` and re-verified by validators.

`receivedAsset(assetName)` is not a first-stage interface. If a future protocol version needs to expose the amount of a specific asset received by the current invoke transaction, that behavior should be specified and implemented separately. The current implementation derives asset inputs and settlement from Call TX outputs, the contract UTXO set, and Result TX validation.

## GAS

EVM Contracts use the unified SatoshiNet GAS asset. GAS pays for contract call cost, VM execution, Result TX packaging, and trigger execution where applicable.

The first stage can use protocol-defined fixed gas pricing. Future versions may introduce more flexible pricing while preserving deterministic validation.

Default GAS limits:

1. The per-deploy/invoke gas limit is `50,000,000`.
2. The per-trigger execution gas limit is `5,000,000`.
3. The per-block EVM execution gas limit is `1,000,000,000`.

Height triggers registered through the SatoshiNet trigger precompile are part of EVM state and therefore part of the EVM state root. A valid trigger registration and execution must satisfy all of the following rules:

1. The trigger gas limit must be positive.
2. The trigger gas limit must not exceed the per-trigger execution gas limit.
3. The contract must have enough GAS funding for trigger execution and Result TX packaging.
4. The accumulated EVM gas used in the block must not exceed the per-block EVM execution gas limit.

## Compatibility Scope

EVM compatibility is a developer-facing execution compatibility layer, not a promise that every Ethereum operational assumption maps directly to SatoshiNet.

Important differences:

1. Contract addresses use SatoshiNet contract-address encoding.
2. Asset settlement is UTXO-backed and canonical-result based.
3. Bitcoin-native assets can enter through SatoshiNet asset flows rather than ERC-only balances.
4. Validators must be able to deterministically replay execution and Result TX construction.

## Roadmap

The EVM path should evolve toward:

1. Stable deployment and invocation ABI conventions.
2. Standard tooling for contract developers.
3. Clear GAS estimation and fee display.
4. Contract explorer support.
5. Safe templates for Bitcoin-native asset handling.

EVM is one smart-contract family in SatoshiNet. It complements template contracts and natural-language contracts rather than replacing them.
