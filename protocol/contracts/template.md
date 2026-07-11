# Template Contracts

Template contracts are SatoshiNet-native smart contracts. Their execution logic is implemented by Go runtimes embedded in SatoshiNet nodes.

Template contracts follow the common [Smart Contract Protocol](./). This page defines only template-specific rules.

## Contract Type

Template contracts use the template contract type. They do not execute EVM bytecode, do not use Solidity ABI, and do not maintain EVM account state. They are not channel contracts and do not depend on RSMC channels, multi-signature peer state, or commitment transactions.

Their asset-control boundary comes from contract VM state and canonical `CONTRACT_RESULT`.

## Address Rules

Template contract addresses use the common format:

```
ca/tc + version + template-contract-type + hash
```

The address hash is 32 bytes and is derived from encoded contract content, deployer, and deployment nonce. The nonce lets the same deployer deploy the same contract content to different addresses.

## Deployment

`CONTRACT_DEPLOY` deploys a template contract. The deployment payload contains the template name, template version, deployer, deployment nonce, gas limit, and encoded contract content.

Nodes derive the contract address, initialize runtime state, and record deployment result and state root through a same-block `CONTRACT_RESULT`. Deploying to an existing contract address is invalid.

## Invocation

`CONTRACT_INVOKE` calls a template contract. The call transaction must output to the called contract address. That output binds the Call TX and Result TX, carries assets sent to the contract, and provides the fee source.

Caller identity is derived from the last input's previous-output address. Runtime user state, LP ownership, refund permissions, and history all use that identity.

Template contracts may define default invocations. A transaction with no contract OP\_RETURN but with an output to a template contract address can trigger contract-defined default behavior.

## Fee and Result Rules

Template contracts use the unified GAS asset. Each template can define its own call fee or matching fee. Fee portions inside gas/funding outputs do not enter the contract asset pool.

Externally submitted `CONTRACT_RESULT` transactions do not enter the mempool. Block producers construct canonical Result TXs from runtime results, and validators independently replay runtime execution and compare the block result.

## State Rules

Template contract state is part of the global contract VM state. Template, EVM, and Agent state roots are combined into the unified contract state root committed in coinbase.

Template runtimes must be deterministic. They cannot depend on local time, external services, random values, or other non-consensus inputs. Contract asset balance must be verifiable from both contract-address UTXOs and runtime state.

## First-Stage Templates

The first-stage templates are:

1. Limit order contract.
2. AMM contract.
3. Asset exchange contract.
4. Autopay contract.

Limit order and AMM templates migrate business logic from earlier channel-contract workflows, but do not keep channel peer signatures, RSMC channel state, L1 deposit/withdraw, L1 commit/reveal or BRC20 transfer inscription flows, or legacy channel-contract state migration.

The Autopay contract is not migrated from channel contracts. It is a first-stage native template for pooling delegated continuous-payment balances from multiple addresses and emitting one unified payment per block.

## Limit Order Contract

The limit order template supports swap and refund actions. Buy orders are sorted from high price to low price, sell orders from low price to high price. Orders at the same price are ordered by block order, transaction order, and item id.

A trade executes when buy price is greater than or equal to sell price. The execution price is the sell price. Result TX outputs transfer the purchased asset to the buyer and sats to the seller. Unused sats return to the buyer. If a sell order remainder cannot be traded at the given price, the remainder returns to the seller.

Refund can cancel selected open orders or all open orders owned by the caller. Filled assets that were already emitted by Result TX do not enter refund again.

Default invocation can interpret sats sent to the contract as buy intent and the contract asset sent to the contract as sell intent when a reference price exists.

## AMM Contract

The AMM template uses a constant-product model. It supports swap, refund, add liquidity, and remove liquidity actions.

Deployment includes asset name, initial asset amount, initial sat amount, and constant `K`. After deployment, the contract becomes tradable only when the contract address has sufficient asset pool, sat pool, and `asset * sats >= K`. If the pool is emptied, the contract leaves tradable state and must be reinitialized through liquidity addition.

AMM buy uses sats as input and outputs assets. AMM sell uses assets as input and outputs sats. Slippage and minimum output are non-economic invocation parameters; actual input assets are derived from the Call TX output to the contract address.

Multiple AMM swaps in the same block are processed sequentially in canonical transaction order. Each swap is priced from the pool and `K` left by the preceding swap. The input amount after the 0.8% service fee participates in the constant-product calculation, while the full input is added to the pool so that the fee remains in the pool. For buys, `Amt` is only the minimum acceptable output; once slippage protection passes, the full input is swapped rather than truncating execution at `Amt` and refunding the remainder. Swaps are processed before add/remove liquidity, matching the channel-contract behavior.

## Asset Exchange Contract

The asset exchange template supports deterministic exchange between two SatoshiNet assets. Its state, price curve, fee rule, and refund rule are encoded in template content and enforced through canonical Result TX.

The exchange template follows the same principles as other templates: assets enter through Call TX outputs, execution is replayed by validators, and asset transfers are settled only by canonical Result TX.

## Autopay Contract

The Autopay template pools delegated continuous-payment balances from multiple addresses and pays one configured recipient, or the current block producer, on each block. Its template name is:

```
autopay.tc
```

Deployment content includes:

1. Service name.
2. Recipient address. If empty, the per-block payment is paid as miner fee to the current block producer.
3. Fee asset name, which can be sats or any SatoshiNet asset.
4. Minimum per-address amount per block.

Deployment content does not include a user list, payment interval, end height, or height-dependent payment curve. Each delegate's amount and balance are written into runtime state by later calls.

Invocation APIs:

1. `config`: a delegate sets its own per-block amount. The amount must not be lower than the contract minimum.
2. `default invoke`: a delegate funds the contract address and increases its own payable balance. If the delegate has not configured an amount, the contract minimum is used.
3. `cancel`: a delegate cancels its own payment configuration and receives its remaining payment balance back.
4. `close`: only the deployer can close the contract and batch-refund remaining balances to all delegates.

The payment asset can be sats or any SatoshiNet asset. If the payment asset is also the GAS asset, runtime must distinguish business payment balance from contract trigger/result gas balance within the same physical asset.

Delegate state:

1. Runtime maintains independent state for each delegate address, including amount per block, funding balance, total paid amount, paid block count, last paid height, and delegate status.
2. Delegate status can be active, funding, or closed.
3. The contract's aggregate payable balance is the sum of all delegate balances.
4. Contract state can expose aggregate balances and per-delegate auditable state.

Activation and funding rules:

1. Deployment and default invocation can both fund the contract address.
2. Funding in the deploy transaction belongs to the deployer.
3. Funding in a default invocation belongs to that invocation's invoker.
4. The contract is active when at least one delegate has enough balance for the next block and the contract has enough trigger/result gas.
5. If a delegate lacks balance for one block, that delegate enters funding status and does not participate in that block's payment. Other sufficiently funded delegates can continue paying.
6. If no delegate can pay, or if the contract lacks required trigger/result gas, the contract enters funding status and waits for more funding.

Payment rules:

1. At each block settlement, runtime scans all active delegates.
2. For each delegate with enough balance, runtime deducts that delegate's per-block amount.
3. The block producer creates one canonical `CONTRACT_RESULT` that aggregates all delegate payments for the block into one payment output.
4. If the recipient address is not empty, the payment output goes to that recipient.
5. If the recipient address is empty, the payment output is treated as miner fee and enters the block reward.
6. One Autopay contract should emit only one payment Result per block, preventing many delegates from producing too many automatic trigger transactions in the block.

Close rules:

1. Only the deployer can close an Autopay contract.
2. On close, the contract refunds each delegate's remaining payment balance to that delegate.
3. Each close Result can contain at most 1000 refund outputs. If the number of delegates exceeds that per-transaction limit, unrefunded balances remain at the contract address and are processed by later Results.
4. After close completes, remaining managed gas balance is returned to the deployer.
5. Assets at the contract address that are not part of the managed balances continue to follow the common framework rule for residual contract-address assets.
