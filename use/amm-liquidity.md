# Provide AMM Liquidity

The AMM in the current PWA Wallet market is an L2 market Channel Contract capability, not a SatoshiNet smart contract. It runs in the existing L2 market system and is used to test AMM pools, swaps, and liquidity operations.

The smart contract template AMM is a separate testing path. It can only be accessed from PWA `Tools -> Smart Contracts` and should not be confused with the market AMM.

## Before Testing

1. Install or open [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. Connect to SatoshiNet testnet.
3. Confirm the wallet has test assets.
4. Confirm the wallet has the test assets and test sats required by the AMM pool.

## Add Liquidity

1. Go to the AMM page in PWA `Market` or DEX.
2. Select a testnet AMM pool.
3. Check the pool assets, current price, liquidity depth, and your balance.
4. Enter the asset amount to add.
5. Enter or confirm the sats amount.
6. Check expected LP share or liquidity position.
7. Confirm the transaction in the wallet.
8. Wait for market transaction state to update.
9. Check whether the wallet and market page show the new liquidity position.

## Swap

1. Select the trading direction.
2. Enter input amount.
3. Check estimated output, slippage, and fees.
4. Confirm the transaction in the wallet.
5. Wait for market transaction state to update.
6. Verify asset balance changes in wallet and Explorer.

If slippage protection fails, the market contract should refund or keep a safe return path for the assets according to Channel Contract rules.

## Remove Liquidity

1. Open your liquidity position.
2. Enter the liquidity share to remove.
3. Check expected returned assets.
4. Confirm the transaction in the wallet.
5. Wait for market transaction state to update.
6. Check whether returned assets appear in the wallet.

## Verification

For every operation, check:

1. Txid.
2. Wallet asset balance.
3. Market contract result and related transaction outputs.
4. Explorer / Indexer state.
5. Whether L2 market state and wallet asset state are consistent.

## Risk Boundary

1. Current market AMM liquidity testing is testnet-only.
2. Testnet prices and liquidity do not represent real market prices.
3. Page display may lag behind chain state.
4. When a network error occurs, first check txid and market state. Do not repeatedly submit the same value movement.

**Page Status: In Development**
