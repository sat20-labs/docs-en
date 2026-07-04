# EVM Sample Contracts

The SatoshiNet EVM Runtime has entered public testnet testing. The current testnet has deployed and verified standard EVM sample contracts to show how Solidity applications run on SatoshiNet EVM Runtime and use SatoshiNet native asset interfaces for asset reading, claiming, and Result TX settlement.

The two standard EVM samples currently on testnet are:

1. `ConstantProductAMM`
2. `LimitOrderBook`

This page documents the current samples from a developer perspective. Testnet contract addresses, txids, Explorer links, and PWA screenshots should be added in later test records.

The sample source is currently at [`sat20wallet/sdk/e2e/testdata/contracts/StandardApps.sol`](https://github.com/sat20-labs/sat20wallet/blob/main/sdk/e2e/testdata/contracts/StandardApps.sol). This source is the reference implementation for testnet standard samples. It does not mean future mainnet contract templates are limited to these two business forms.

## Current Samples

| Sample | Type | What It Verifies |
| --- | --- | --- |
| `ConstantProductAMM` | Solidity AMM sample | add liquidity, swap, remove liquidity, asset precompile, ABI calldata, Result TX |
| `LimitOrderBook` | Solidity limit order sample | create order, fill order, cancel order, order state view, asset precompile, ABI calldata, Result TX |

If testnet samples change later, update this page and [EVM Developer Preview](evm-quickstart.md).

## ConstantProductAMM

`ConstantProductAMM` is a constant-product AMM sample contract on EVM Runtime. It implements an AMM state machine in Solidity and uses the SatoshiNet asset precompile to handle real asset inputs and outputs.

Core interfaces include:

| Interface | Purpose |
| --- | --- |
| `addLiquidity(uint256 minLiquidity)` | Add asset and sats liquidity, mint internal LP share |
| `swapSatForAsset(string minAssetOut)` | Input sats and output target asset |
| `swapAssetForSat(uint256 minSatOut)` | Input target asset and output sats |
| `removeLiquidity(uint256 liquidity, string minAssetOut, uint256 minSatOut)` | Remove LP share and withdraw asset and sats |
| `liquidityOfCaller()` | Query the current caller's LP share |
| `reserves()` | Query pool asset, asset reserve, sats reserve, and total LP |
| `quoteSatForAsset(uint256 satIn)` | Estimate asset output for sats input |
| `quoteAssetForSat(string assetIn)` | Estimate sats output for asset input |
| `quoteAddLiquidity(string assetIn, uint256 satIn)` | Estimate LP share minted by adding liquidity |
| `quoteRemoveLiquidity(uint256 liquidity)` | Estimate asset and sats output for removed liquidity |

Test goals:

1. Deploy the Solidity AMM contract, with constructor parameter specifying the managed asset name.
2. Let PWA generate ABI calldata from function signature and arguments.
3. Input assets and sats through funding output; these economic inputs are not duplicated in Solidity parameters.
4. The contract calls `fundingAssetAmount`, `fundingSats`, and `claimFundingAsset` to identify and claim input assets.
5. The contract updates reserve and LP state.
6. During swap, the contract emits asset transfer intents through `transferAsset`.
7. SatoshiNet completes output settlement through canonical Result TX.
8. `reserves()` and Explorer / Indexer contract asset state remain explainably consistent.

`ConstantProductAMM` is similar in purpose to the template AMM contract, but it verifies the Solidity / EVM path: developers can express AMM logic in EVM contracts while asset settlement is still handled by the SatoshiNet UTXO asset layer and Result TX.

## LimitOrderBook

`LimitOrderBook` is a limit order sample contract on EVM Runtime. It maintains order state in Solidity and uses the SatoshiNet asset precompile to process order creation, filling, cancellation, and asset settlement.

Core interfaces include:

| Interface | Purpose |
| --- | --- |
| `createOrder(string sellAsset, string buyAsset, string buyAmount)` | Create an order; the sell asset comes from the current funding output |
| `fillOrder(uint256 orderId)` | Fill an order; payment asset comes from the current funding output |
| `cancelOrder(uint256 orderId)` | Maker cancels an open order and withdraws remaining sell asset |
| `activeOrderCount()` | Query active order count |
| `activeOrderId(uint256 activeIndex)` | Query order ID by active index |
| `orderInfo(uint256 orderId)` | Query order state |
| `stateView()` | Return order-book JSON view |
| `quoteFillOrder(uint256 orderId, string paidIn)` | Estimate sell asset output for a fill amount |

Core flow:

1. Maker creates an order and transfers the sell asset to the contract address.
2. PWA generates ABI calldata from `createOrder(string,string,string)`; `buyAmount` in calldata is the expected buy asset amount, while sell asset amount comes from funding output.
3. The contract reads funding output and confirms the sell asset amount.
4. The contract records order state, including maker, recipient address, sell asset, buy asset, remaining sell amount, and remaining buy amount.
5. Taker calls `fillOrder(uint256)` and transfers the required buy asset through funding output.
6. The contract calculates fill ratio and updates remaining order state.
7. The contract transfers payment asset to maker and sell asset to taker through `transferAsset`.
8. Maker can cancel an open order, and the contract refunds remaining sell asset through Result TX.

Test goals:

1. Verify that an EVM contract can maintain order-book state.
2. Verify that maker sell asset is provided by funding output, not by the amount in calldata.
3. Verify that taker fill asset is provided by funding output.
4. Verify that partial fill, full fill, and cancellation can all settle through Result TX.
5. Verify consistency among order state view, wallet balance, and Explorer / Indexer evidence.

`LimitOrderBook` is similar in purpose to the template limit order contract, but it verifies the Solidity / EVM path: order-book state is expressed in EVM storage, and asset transfer is still settled by SatoshiNet native asset interfaces and canonical Result TX.

## Asset Precompile

EVM contracts cannot spend UTXOs directly. They use the SatoshiNet asset precompile to express asset reads and transfer intents.

Common interfaces include:

| Interface | Purpose |
| --- | --- |
| `balanceOf(address,string)` | Query the specified asset balance for the EVM-address-derived contract address |
| `fundingAssetAmount(string)` | Query the specified asset amount in the current Call TX funding output |
| `claimFundingAsset(string,string)` | Claim a specified asset amount from current funding for business processing |
| `fundingSats()` | Query the sats amount transferred to the contract address in the current Call TX |
| `callerAddress()` | Query the caller's SatoshiNet receiving address, used for refund and asset ownership |
| `transferAsset(string,string,string,bytes)` | Declare one asset transfer intent |
| `transferAssets(string[],string[],string[],bytes[])` | Declare multiple asset transfer intents in one call |
| `compareAmount(string,string)` | Compare amounts with Decimal semantics |

See [EVM Contracts](../protocol/contracts/evm.md) for the full rules.

## Testnet Operation Path

Current recommended path:

1. Open [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. Confirm that it is connected to SatoshiNet testnet.
3. Claim test GAS from the `Tools` home page.
4. Go to the EVM contract testing or deployment tool under PWA `Tools -> Smart Contracts`.
5. Use the Solidity source compilation entry, or a standard sample entry provided by PWA, to prepare init code for `ConstantProductAMM` or `LimitOrderBook`.
6. Deploy the contract and wait for Deploy TX and same-block Result TX.
7. Before invoking a sample interface, enter function signature, arguments, sats, funding assets, and gas limit, then generate calldata and complete estimate.
8. Confirm that calldata matches the current parameters, then sign and broadcast.
9. Check Invoke TX, Result TX, state view, and asset changes in wallet, tools page, or Explorer.

If PWA has not opened the full source deployment flow for ordinary users, use the standard sample entry first. CLI, RPC, sample repository, and a fuller Solidity developer flow remain in public testnet iteration.

## Verification Checklist

Each EVM sample should provide:

1. Deploy TX.
2. Contract address.
3. Invoke TX.
4. Result TX, if the invocation produces asset transfer, refund, revert, or out of gas.
5. Contract state change.
6. The block containing contract state root.
7. Contract metadata, including source, ABI, compiler config, and code hash.
8. Calldata generation record, plus sats, funding assets, and gas limit for the invocation.

## Limitations

1. These are testnet samples and do not represent mainnet availability.
2. Testnet parameters, tool entries, and sample contracts may change.
3. EVM internal balances are not the same as SatoshiNet native asset balances.
4. Function parameters should not duplicate economic inputs already expressed by funding output.
5. Real asset transfer must be verified through Result TX.
6. Solidity contracts must follow SatoshiNet's UTXO asset model and asset precompile rules.

**Page Status: In Development**
