# Template Contracts

Template contracts are SatoshiNet-native smart contracts. Their execution logic is implemented by Go runtimes embedded in SatoshiNet nodes.

Template contracts follow the common [Smart Contract Protocol](readme.md). This page defines only template-specific rules.

## Contract Type

Template contracts use the template contract type. They do not execute EVM bytecode, do not use Solidity ABI, and do not maintain EVM account state. They are not channel contracts and do not depend on RSMC channels, multi-signature peer state, or commitment transactions.

Their asset-control boundary comes from contract VM state and canonical `CONTRACT_RESULT`.

## Address Rules

Template contract addresses use the common format:

```text
ca/tc + version + template-contract-type + hash
```

The address hash is 32 bytes and is derived from encoded contract content, deployer, and deployment nonce. The nonce lets the same deployer deploy the same contract content to different addresses.

## Deployment

`CONTRACT_DEPLOY` deploys a template contract. The deployment payload contains the template name, template version, deployer, deployment nonce, gas limit, and encoded contract content.

Nodes derive the contract address, initialize runtime state, and record deployment result and state root through a same-block `CONTRACT_RESULT`. Deploying to an existing contract address is invalid.

## Invocation

`CONTRACT_INVOKE` calls a template contract. The call transaction must output to the called contract address. That output binds the Call TX and Result TX, carries assets sent to the contract, and provides the fee source.

Caller identity is derived from the last input's previous-output address. Runtime user state, LP ownership, refund permissions, and history all use that identity.

Template contracts may define default invocations. A transaction with no contract OP_RETURN but with an output to a template contract address can trigger contract-defined default behavior.

## Fee and Result Rules

Template contracts use the unified GAS asset. Each template can define its own call fee or matching fee. Fee portions inside gas/funding outputs do not enter the contract asset pool.

Externally submitted `CONTRACT_RESULT` transactions do not enter the mempool. Block producers construct canonical Result TXs from runtime results, and validators independently replay runtime execution and compare the block result.

## State Rules

Template contract state is part of the global contract VM state. Template, EVM, and Agent state roots are combined into the unified contract state root committed in coinbase.

Template runtimes must be deterministic. They cannot depend on local time, external services, random values, or other non-consensus inputs. Contract asset balance must be verifiable from both contract-address UTXOs and runtime state.

## First-Stage Templates

The first-stage templates are transaction-oriented:

1. Limit order contract.
2. AMM contract.
3. Asset exchange contract.

Limit order and AMM templates migrate business logic from earlier channel-contract workflows, but do not keep channel peer signatures, RSMC channel state, L1 deposit/withdraw, L1 commit/reveal or BRC20 transfer inscription flows, or legacy channel-contract state migration.

## Limit Order Contract

The limit order template supports swap and refund actions. Buy orders are sorted from high price to low price, sell orders from low price to high price. Orders at the same price are ordered by block order, transaction order, and item id.

A trade executes when buy price is greater than or equal to sell price. The execution price is the sell price. Result TX outputs transfer the purchased asset to the buyer and sats to the seller. Unused sats return to the buyer. If a sell order remainder cannot be traded at the given price, the remainder returns to the seller.

Refund can cancel selected open orders or all open orders owned by the caller. Filled assets that were already emitted by Result TX do not enter refund again.

Default invocation can interpret sats sent to the contract as buy intent and the contract asset sent to the contract as sell intent when a reference price exists.

## AMM Contract

The AMM template uses a constant-product model. It supports swap, refund, add liquidity, and remove liquidity actions.

Deployment includes asset name, initial asset amount, initial sat amount, and constant `K`. After deployment, the contract becomes tradable only when the contract address has sufficient asset pool, sat pool, and `asset * sats >= K`. If the pool is emptied, the contract leaves tradable state and must be reinitialized through liquidity addition.

AMM buy uses sats as input and outputs assets. AMM sell uses assets as input and outputs sats. Slippage and minimum output are non-economic invocation parameters; actual input assets are derived from the Call TX output to the contract address.

Multiple AMM swaps in the same block are priced from the pool state and `K` at the start of that block's settlement. Swaps with the same direction and same input amount should receive the same quoted output. Actual settlement is still bounded by the pool assets available in that block. Within a block, swaps are processed before add/remove liquidity, matching the earlier channel-contract behavior.

## Asset Exchange Contract

The asset exchange template supports deterministic exchange between two SatoshiNet assets. Its state, price curve, fee rule, and refund rule are encoded in template content and enforced through canonical Result TX.

The exchange template follows the same principles as other templates: assets enter through Call TX outputs, execution is replayed by validators, and asset transfers are settled only by canonical Result TX.
