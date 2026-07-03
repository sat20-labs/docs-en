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

EVM calldata is passed directly to the EVM as the call input. The protocol layer does not predefine business actions or business parameters for EVM contracts. Method selectors, deadlines, slippage, or other non-economic parameters can be carried in calldata. Economic parameters are derived from transaction outputs to the contract address. If a contract needs to know the assets carried by the current Call TX, it should read the funding output through the native asset precompile instead of asking the user to duplicate the same economic value in calldata.

## Asset Boundary

The EVM account model and the SatoshiNet UTXO model must stay consistent:

1. EVM execution decides logical contract state.
2. SatoshiNet UTXOs represent actual asset inputs and outputs.
3. Asset transfers caused by EVM execution are settled by canonical `CONTRACT_RESULT`.
4. Validators replay EVM execution and reject blocks whose Result TX does not match replayed output.

This avoids treating off-chain or non-canonical EVM traces as final asset transfers.

## Native Asset Precompile

EVM contracts cannot spend UTXOs directly. They can only query contract asset state and emit asset-transfer intents through the SatoshiNet native asset precompile.

The current asset precompile address is:

```text
0x0000000000000000000000000000000000534E01
```

The first-stage asset precompile interface is:

1. `balanceOf(address owner, string assetName) returns (string)`: returns the available balance of the specified asset for the contract address derived from the EVM address.
2. `fundingAssetAmount(string assetName) returns (string)`: returns the remaining claimable amount of the specified asset in the current Call TX funding output. If `assetName` is the unified GAS asset, the amount is returned after reserving the Result fee for this call.
3. `fundingSats() returns (uint256)`: returns the plain sats amount in the current Call TX funding output.
4. `claimFundingAsset(string assetName, string amount) returns (bool)`: explicitly claims an amount from the current funding output for business processing.
5. `callerAddress() returns (string)`: returns the SatoshiNet caller address derived by the common caller rule.
6. `transferAsset(string assetName, string to, string amount, bytes extraData) returns (bool)`: declares one asset-transfer intent.
7. `transferAssets(string[] assetNames, string[] recipients, string[] amounts, bytes[] extraData) returns (bool)`: declares multiple asset-transfer intents. This is the preferred interface for close and batch settlement.
8. `compareAmount(string left, string right) returns (int256)`: compares two asset amount strings with Decimal semantics.
9. `addAmount`, `subAmount`, `mulAmount`, and `divAmount`: perform Decimal arithmetic on asset amount strings.
10. `uintToAmount(uint256)`, `amountToUintFloor(string)`, and `amountToUintCeil(string)`: convert between integer values and asset amount strings.

Asset amounts in the precompile interface are string-first. A Solidity contract may convert to `uint256` for internal calculations, but final transfer intents must return to asset amount strings and are still checked against asset precision by Result TX validation.

## Contract Metadata and State View

To let wallets, markets, and explorers show useful information without knowing the full Solidity source, EVM contracts should implement a small read-only base interface:

```solidity
function contractName() external view returns (string memory);
function contractSubtype() external view returns (string memory);
function managedAssetCount() external view returns (uint256);
function managedAsset(uint256 index) external view returns (string memory);
function managedAssetBalance(uint256 index) external view returns (string memory);
function managedAssetBalance(string calldata assetName) external view returns (string memory);
```

Nodes try to call these methods when building an EVM state view. If `contractName()` is unavailable, the display name is `unknown`. The managed-asset methods return the assets and amounts that the contract itself believes it manages. Explorers may also show the actual UTXO-backed balance at the contract address; the two views are related but not required to be identical.

Contracts can also implement:

```solidity
function stateView() external view returns (string memory);
```

`stateView()` returns a contract-defined JSON string for order books, pools, user shares, or other business-specific views. It is a display projection, not a replacement for consensus state.

## Deployment Tooling and Source Metadata

Wallets may provide a Solidity source compilation flow. Compiler parameters are defined by the SatoshiNet-supported configuration instead of being freely chosen by users. The current test-stage defaults are:

1. `solcVersion`: `0.8.30`.
2. `evmVersion`: `paris`.
3. optimizer enabled with `runs=200`.
4. metadata `bytecodeHash=none`.
5. single-file source only; imports are not allowed.

After deployment confirms, wallets or deployment tools can submit contract address, Solidity source, ABI, compiler config, constructor arguments, and init/runtime code hashes to the L2 indexer metadata API. This metadata helps wallets, markets, and explorers display and encode calls. It is not consensus state and does not replace the on-chain deployment transaction, code hash, or state root.

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

## Close Rule

EVM contracts use the common smart-contract close rule. When the deployer closes a contract, the EVM executor first attempts to call:

```solidity
function close() external returns (bool);
```

The contract should use the native asset precompile to emit transfer intents for assets with clear ownership, such as LP shares, open orders, deposits, or other user positions. After `close()` finishes, the framework applies the common close handling for remaining contract-managed profit and unmanaged assets held at the contract address.

Public EVM contracts should implement an explicit `close()` and use `transferAssets` when close needs to distribute multiple assets to multiple recipients.
