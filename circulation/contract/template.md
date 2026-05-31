Template Contracts
====

Template contracts are native SatoshiNet smart contracts. Their execution logic is implemented by Go runtimes embedded in SatoshiNet node source code.

Template contracts follow the common [Smart Contracts](readme.md) protocol. This document defines template-specific rules only.


Contract Type
----

Template contract type:

```text
ContractTypeTemplate
```

Template contracts do not execute EVM bytecode, do not use Solidity ABI, and do not maintain EVM account state. They are not channel contracts and do not depend on RSMC channels, multi-party signatures, or commitment transactions.

The asset-control boundary of a template contract is contract VM state plus canonical `CONTRACT_RESULT`.


Address Rules
----

Template contracts use the common contract address format:

```text
ca/tc + version + ContractTypeTemplate + hash
```

The template contract address hash is 32 bytes. Hash input:

1. Contract content returned by `Contract.Encode()`.
2. deployer.
3. deploy random value.

The random value allows the same deployer to deploy the same contract content to different contract addresses.

`URL()` in the template runtime equals the contract address.


Deployment
----

`CONTRACT_DEPLOY` deploys a template contract.

Deploy payload contains:

1. template name.
2. template version.
3. deployer.
4. deploy random value.
5. gas limit.
6. contract content returned by `Contract.Encode()`.

Nodes derive the contract address from the payload, initialize runtime state, and record deployment result and state root through `CONTRACT_RESULT` in the same block.

If the derived contract address already exists, deployment is invalid.


Invocation
----

`CONTRACT_INVOKE` calls a template contract.

The call transaction must output to the invoked contract address. That output is used as:

1. The binding UTXO between Call TX and Result TX.
2. Assets transferred into the contract by this call.
3. The fee source for this call and later settlement.

Invoker identity is resolved from the last input of the call transaction. Runtime user state, LP ownership, refund permission, and history records use this identity.

Template contracts may define default invocations. A default invocation is triggered by a transaction that has no contract OP_RETURN but sends an output to a template contract address. The contract interprets the business meaning from the asset type and amount in that output. If the output does not satisfy executable contract conditions, the contract may treat it as invalid business input without changing business state.


Fees
----

Template contracts use the unified gas asset.

Each template may define its own call fee and matching fee. The limit order and AMM templates keep the fee calculation style of the previous channel contracts, but the fee asset is the unified gas asset.

The fee portion of a gas/funding output does not enter the contract asset pool. Only the remaining assets after fee deduction become contract-controlled assets.


Result TX
----

Template contracts do not accept externally submitted `CONTRACT_RESULT` transactions in mempool.

The block producer constructs canonical `CONTRACT_RESULT` from template runtime results. Validating nodes independently replay the runtime and compare the Result TX in the block.

If Result TX is missing, non-canonical, or inconsistent with local replay, the block is invalid.


State
----

Template contract state is part of the global contract VM state.

The template state root, EVM state root, and Agent state root are combined into one contract state root and written into coinbase.

Template runtimes must:

1. Produce the same state for the same on-chain input.
2. Not depend on local time, external services, randomness, or other non-deterministic data.
3. Validate contract asset balances using both contract-address UTXOs and runtime state.


First-Phase Templates
----

The first phase implements three trading templates:

1. Limit order trading contract.
2. AMM trading contract.
3. Asset exchange contract.

The limit order and AMM templates migrate business logic from previous channel contracts, but remove:

1. channel-side signatures.
2. RSMC channel state.
3. L1 deposit/withdraw.
4. L1 commit/reveal and BRC20 transfer inscription flows.
5. legacy channel-contract state migration.


Limit Order Trading Contract
----

The limit order trading contract corresponds to the limit order logic of the previous `SwapContractRuntime`.

API names and order type values remain compatible with the channel contract:

1. `swap`: place a buy or sell order.
2. `refund`: cancel orders or retrieve refundable assets.
3. buy order type is `2`.
4. sell order type is `1`.
5. refund order type is `3`.

Matching rules:

1. Buy orders are sorted by price descending.
2. Sell orders are sorted by price ascending.
3. Same-price orders are sorted by block order, transaction order, and item id.
4. A trade is valid only when buy price is greater than or equal to sell price.
5. Trade price is the sell order price.
6. A buy order may fill against lower-price sell orders.
7. Each match produces an asset transfer to the buyer and a satoshi transfer to the seller in the same Result TX.
8. When a buy order is filled, unused satoshis are returned to the buyer.
9. When remaining sell asset is too small to trade at the order price, it is returned to the seller.

Refund rules:

1. `refund` takes a list of item ids.
2. An empty item id list cancels all unfinished orders of the invoker.
3. A non-empty list cancels only the specified items.
4. The invoker can cancel only their own orders.
5. Assets already sent out by Result TX are not refundable.

Default invocation rules:

1. Sending satoshis to a limit order contract is interpreted as a buy at the currently executable sell price.
2. Sending the contract asset to a limit order contract is interpreted as a sell at the currently executable buy price.
3. If no reference trade price exists, the default invocation creates no new order and produces no asset settlement.


AMM Trading Contract
----

The AMM trading contract corresponds to the swap and liquidity logic of the previous AMM channel contract.

API names and order type values remain compatible with the channel contract:

1. `swap`: AMM buy or sell.
2. `refund`: retrieve refundable assets.
3. `addliq`: add liquidity.
4. `removeliq`: remove liquidity.
5. buy order type is `2`.
6. sell order type is `1`.
7. add-liquidity order type is `9`.
8. remove-liquidity order type is `10`.

Deployment and ready rules:

1. AMM deploy content contains asset name, initial asset amount, initial satoshi amount, and constant K.
2. After deployment, the contract is not tradable unless the contract address holds enough asset, satoshi, and `asset*sats >= K`.
3. `addliq` can be used to fill the initial pool.
4. The contract becomes tradable when asset pool, satoshi pool, and `asset*sats >= K` are all satisfied.
5. After the contract is ready, normal trading does not recheck real-time `asset*sats >= K`.
6. If the pool becomes empty, the contract exits tradable state and must reach the initial K again through `addliq`.

AMM swap rules:

1. AMM uses the constant product formula.
2. AMM buy: user inputs satoshis and receives assets.
3. AMM sell: user inputs assets and receives satoshis.
4. In AMM sell, `Amt` is the minimum acceptable output satoshi amount. Input asset amount comes from the Call TX funding output.
5. In AMM buy, `Amt` is the minimum acceptable output asset amount. Input satoshi amount is determined by `UnitPrice` and the funding output.
6. Slippage protection failure generates a refund result in the same block.
7. Within the same block, AMM processes swaps before add/remove liquidity, matching the previous channel contract.
8. If `addliq` makes the contract ready in a block, pending swaps are matched in later blocks.

AMM default invocation rules:

1. Sending satoshis to an AMM contract is interpreted as a buy.
2. Sending the contract asset to an AMM contract is interpreted as a sell.
3. A default invocation that does not match AMM trading semantics does not change AMM business state.

Liquidity rules:

1. `addliq` contains asset amount and satoshi amount to enter the pool.
2. The satoshi amount entering the pool is the `Value` field in `addliq`, not all sats in the Call TX funding output.
3. Excess asset or satoshi is returned according to the pool ratio.
4. `removeliq` contains the LPT amount to remove.
5. If requested LPT exceeds the invoker balance, the actual invoker balance is used.
6. Removing liquidity returns the corresponding pool share to the LP.
7. If profit exists, LP receives 60% of profit. The previous server share and foundation share are merged and paid to the foundation.
8. The current implementation uses deployer address as the foundation recipient.


Asset Exchange Contract
----

The asset exchange contract sells inventory asset A according to deployer-defined rules. The invoker inputs asset B and receives asset A. The deployer receives the consumed asset B in the same canonical `CONTRACT_RESULT`.

Deployment content contains:

1. asset A name.
2. asset B name.
3. price mode.
4. price step table.

Price modes:

1. `height`: choose the current price step by block height.
2. `sold_a`: choose the current price step by cumulative sold asset A amount.

In the price step table, `bPerA` means the amount of asset B required for 1 unit of asset A. The first threshold must be 0, and later thresholds must be strictly increasing.

API rules:

1. `exchange`: input asset B and exchange for asset A. The parameter may contain the minimum acceptable output amount of asset A.
2. `close`: only the deployer can call it. It closes the contract and withdraws remaining asset A.
3. A parameterless default invocation with asset A is treated as inventory funding.
4. A parameterless default invocation with asset B is treated as an exchange at the current price.
5. A parameterless default invocation with both asset A and asset B first adds asset A to inventory, then uses asset B for exchange.

Exchange rules:

1. Trade price is determined by the deploy-time price mode and current block state.
2. The contract can output at most the currently available inventory asset A.
3. If inventory asset A is insufficient for the input asset B, the contract partially fills at the current price and refunds unused asset B to the invoker.
4. If the minimum output asset A in the `exchange` parameter is not satisfied, net input asset B is refunded to the invoker.
5. After the contract is closed, it no longer accepts new exchanges and later exchange inputs fail.

Gas-name rules:

1. Asset A or asset B may have the same name as the unified gas asset.
2. If the gas asset is asset A, the Result fee is deducted from available asset A first. Only the remainder can be inventory or exchange output.
3. If the gas asset is asset B, the Result fee is deducted from the current input asset B first. Only the remainder participates in exchange.
4. If the gas asset differs from both asset A and asset B, gas follows the common template contract fee rules.
