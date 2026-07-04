# Use Limit Orders

The limit order feature in the current PWA Wallet market is an L2 market Channel Contract capability, not a SatoshiNet smart contract. It is used to test order creation, filling, cancellation, and asset settlement in the market.

The smart contract template LimitOrder is a separate testing path. It can only be accessed from PWA `Tools -> Smart Contracts` and should not be confused with market limit orders.

## Before Testing

1. Install or open [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. Connect to SatoshiNet testnet.
3. Confirm the wallet has test assets and test sats.
4. Understand the asset pair, price, amount, and fee rules for the order.

## Create an Order

1. Go to the limit order page in PWA `Market` or DEX.
2. Select the trading pair.
3. Choose buy or sell.
4. Enter order price.
5. Enter order amount.
6. Check total input and expected output.
7. Confirm the transaction in the wallet.
8. Wait for market transaction state to update.
9. Check whether the order appears in open orders.

## Fill an Order

1. Select an open order.
2. Enter fill amount.
3. Check expected received asset and paid asset.
4. Confirm the transaction in the wallet.
5. Wait for market transaction state to update.
6. Check wallet balance changes.

## Cancel an Order

1. Open your active orders.
2. Select the order to cancel.
3. Confirm cancellation in the wallet.
4. Wait for market transaction state to update.
5. Check whether remaining locked assets return to your wallet.

## Verification

1. Whether order creation txid exists.
2. Whether the order appears in market state.
3. Whether fill txid and asset output match the expected price.
4. Whether market contract result transfers filled assets to both sides.
5. Whether cancelled assets return to the maker.
6. Whether wallet, market page, and Explorer are consistent.

## Risk Boundary

1. Current limit order testing is testnet-only.
2. Testnet orders and prices do not represent real market value.
3. Page state may lag behind chain state.
4. If a transaction result is unknown, first check txid and contract state. Do not repeatedly submit the same order or fill.

**Page Status: In Development**
