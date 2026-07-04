# Deploy Limit Order Module

The LimitOrder template is one of the smart contract templates currently live on the SatoshiNet public testnet. It validates how order creation, filling, cancellation, and refund are represented by canonical Result TX and contract state root in the SatoshiNet smart contract framework.

Note: The limit order feature in PWA `Market` or DEX is an L2 market Channel Contract capability, not the smart contract template LimitOrder discussed here. The template LimitOrder in this page can only be accessed from PWA `Tools -> Smart Contracts`.

## Current Status

| Item | Status |
| --- | --- |
| Contract type | Template Contract |
| Template | LimitOrder |
| Environment | SatoshiNet public testnet |
| Mainnet status | Not open yet |
| Fee asset | Test GAS |
| Interaction entry | PWA `Tools -> Smart Contracts` |

## Basic Flow

1. The deployer enters PWA `Tools -> Smart Contracts`.
2. Select trading asset and base asset.
3. Deploy the limit order template contract.
4. Maker creates an order and transfers the sell asset to the contract address.
5. Taker fills the order and transfers payment asset to the contract address.
6. The contract matches according to price and order rules.
7. Result TX sends traded assets to both parties.
8. Unfilled or cancelled parts are returned through refund.

## Invocation Actions

| Action | Description |
| --- | --- |
| `swap` / create order | Place buy or sell order; direction is determined by input asset and order parameters |
| `refund` | Cancel order or withdraw refundable assets |

Order types follow the template protocol definition:

| Type | Meaning |
| --- | --- |
| `1` | Sell order |
| `2` | Buy order |
| `3` | refund |

## Matching Rules Summary

1. Buy orders are sorted from high price to low price.
2. Sell orders are sorted from low price to high price.
3. Same-price orders are sorted by block order, transaction order, and item id.
4. A trade happens only when buy price is greater than or equal to sell price.
5. Trade price uses the sell order price.
6. Each match generates buyer asset transfer and seller sats transfer, both in the same Result TX.
7. Unused sats from a buy order are returned to the buyer.
8. When remaining sell asset is insufficient for further matching, the remainder is returned to the seller.

## Verification Checklist

1. Whether Deploy TX created the limit order contract address.
2. Whether order creation transaction transferred sell asset to the contract address.
3. Whether the order entered open state.
4. Whether fill transaction transferred the correct payment asset.
5. Whether Result TX distributed assets by matching rules.
6. Whether refund can only cancel the caller's own open order.
7. Whether order state, Result TX, and wallet balance are consistent.

## Risk Boundary

1. The current limit order template contract is testnet-only.
2. Testnet orders, prices, and assets have no mainnet value.
3. Order state should be interpreted together with wallet, Explorer, and Indexer evidence.
4. When network errors or page delays occur, do not resubmit the same value movement. Check txid and contract state first.

**Page Status: In Development**
