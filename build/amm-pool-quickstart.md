# Deploy AMM Pool

The AMM template is one of the smart contract templates currently live on the SatoshiNet public testnet. It validates AMM logic under the SatoshiNet smart contract framework with whole-network consensus execution, canonical Result TX, and contract state root.

Note: The AMM in PWA `Market` or DEX is an L2 market Channel Contract capability, not the smart contract template AMM discussed here. The template AMM in this page can only be accessed from PWA `Tools -> Smart Contracts`.

## Current Status

| Item | Status |
| --- | --- |
| Contract type | Template Contract |
| Template | AMM |
| Environment | SatoshiNet public testnet |
| Mainnet status | Not open yet |
| Fee asset | Test GAS |
| Interaction entry | PWA `Tools -> Smart Contracts` |

## Basic Flow

1. The deployer selects the trading asset.
2. Select the AMM template in PWA `Tools -> Smart Contracts`.
3. Set initial asset amount, initial sats amount, and constant K.
4. Deploy the AMM template contract.
5. Add initial liquidity to the contract so that asset pool, sats pool, and `asset * sats >= K` satisfy ready conditions.
6. Invoke `swap` to buy or sell.
7. Invoke `addliq` or `removeliq` to adjust liquidity.
8. The block producer generates canonical Result TX.
9. Wallet, Explorer, and Indexer show pool state, trade result, and LP share.

## Deployment Parameters

AMM template deployment should at least include:

| Parameter | Meaning |
| --- | --- |
| Asset name | Pool trading asset, using SatoshiNet asset name format |
| Initial asset amount | Initial asset amount entering the pool |
| Initial sats amount | Initial satoshi amount entering the pool |
| K | Initial constant-product constraint |

Before testnet deployment, check:

1. The wallet has test GAS.
2. The target asset is available on SatoshiNet.
3. Initial asset and initial sats can enter the contract address.
4. Initial K matches liquidity size.
5. The deployer understands that testnet may restart or roll back.

## Invocation Interfaces

Current core actions of the AMM template:

| Action | Description |
| --- | --- |
| `swap` | Buy or sell |
| `refund` | Withdraw refundable assets |
| `addliq` | Add liquidity |
| `removeliq` | Remove liquidity |

Input amount is determined by the funding output transferred to the contract address in the Call TX. Invocation parameters should only express direction, minimum acceptable output, slippage constraint, deadline, or other values that cannot be inferred from funding output.

## Verification Checklist

1. Whether Deploy TX created the AMM contract address.
2. Whether initial liquidity entered the contract address.
3. Whether the contract entered ready state.
4. Whether swap invocation generated the expected Result TX.
5. Whether Result TX sent output asset to the user.
6. Whether LP share and pool balance changed after adding or removing liquidity.
7. Whether Explorer / Indexer / wallet display is consistent.

## Risk Boundary

1. The current AMM template contract is testnet-only.
2. Pool price is determined by test assets and test liquidity and does not represent real market price.
3. Testnet confirmation, indexing, and Result TX display may be delayed.
4. Without Result TX or state evidence, page balance should not be treated as final result.

**Page Status: In Development**
