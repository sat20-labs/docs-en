# Complete Your First Swap

The PWA `Market` entry selects the L2 market from the current wallet network: [satsnet.ordx.market](https://satsnet.ordx.market/swap/?network=mainnet) for mainnet and [test-satsnet.ordx.market](https://test-satsnet.ordx.market/swap/?network=testnet) for testnet. Current acceptance covers AMM and limit orders. Market Channel Contracts are different from template AMM/LimitOrder under `Tools -> Smart Contracts`.

## Flow

1. Confirm the PWA network and open `Market`. The URL must carry the same `network` value.
2. Market requests accounts or transactions through SAT20 DApp Connect; the wallet accepts only allowed origins and unexpired, non-duplicate request IDs.
3. Select assets and review price, slippage, fee, and estimated result.
4. In the wallet approval page, verify origin, network, assets, amount, price/minimum output, and fee. Rejecting must terminate the request without repeated prompts.
5. After approval, wait for both the transaction and market item to reach a terminal state. A toast or txid alone is not proof of settlement.
6. Verify input, output, change, miner fee, and contract state on the asset page.

## Minimum Coverage

| Type | Minimum Validation |
| --- | --- |
| AMM | Read an existing pool, execute one feasible swap, and verify balances/reserves/results; add/remove liquidity when in scope |
| Limit order | List the amount carried by one UTXO, fill from another wallet, verify both parties, then cancel/refund an unfilled order |

When 100 or 1000 units exceed pool depth or price constraints, calculate a feasible amount such as 10 instead of blind repeated retries.

**Page Status: Implemented / Testnet Iterating**
