# Wallet and Assets: First Entry into SatoshiNet

This page describes the current SAT20 PWA Wallet entries, asset locations, and cross-layer operations. The PWA supports mainnet and testnet; test features, Faucet, and contract drills must remain on testnet.

## Preparation

1. Open or install [SAT20 PWA Wallet](https://sat20.org/pwa/).
2. Create or import a wallet, back up the seed phrase, and unlock it.
3. Confirm network, wallet, and subaccount.
4. Prepare Bitcoin L1 assets or test assets and reserve network fees. Testnet SGAS is available under `Tools -> Faucet`.

## Current Wallet Entries

| Entry | Current Capability |
| --- | --- |
| Assets | Balances and send/receive across Bitcoin L1, SatoshiNet L2, and Channel locations |
| Channel mode | Open, close, splicing-in/out, unlock, lock, and user-approved lock-with-expand when capacity is insufficient |
| Market | Opens the network-matched AMM/limit-order market in a controlled embedded page and approves requests in the wallet |
| Tools | Test Faucet, smart-contract deploy/invoke, Mint, and Mining entries |
| Settings | Subaccounts, account management, operation logs, password, phrase, public key, UTXO, node, and referrer |
| RGB11 | L1 issue, import, invoice, send, and account recovery; STP transfer into SatoshiNet is not yet open |

## Standard Flow

1. Review L1 assets and UTXOs. Existing trusted cache should remain visible if a remote refresh fails.
2. Review SatoshiNet assets. A normal L1 or L2 send is submitted only after real broadcast.
3. Enter Channel mode. Before open, the wallet reads server fee configuration and displays capacity, service fee, and reserved network fee.
4. After funding broadcast, monitoring continues until confirmation; short UI timeouts must not turn a broadcast lifecycle into a retryable failure.
5. Use splicing-in to add L1 assets to the channel and unlock to release them to an L2 user address. Return through lock, splicing-out, or close.
6. If lock lacks capacity, the wallet explains the extra Bitcoin fee and asks before lock-with-expand.
7. Verify the final state in the wallet, Bitcoin Explorer, SatoshiNet Explorer, and Indexer.

For pending operations, inspect operation logs and reservation/transaction state before any retry. Never send a seed phrase to an Agent or web form.

**Page Status: Implemented / Iterating; checked against PWA and Wallet SDK source on 2026-08-29**
