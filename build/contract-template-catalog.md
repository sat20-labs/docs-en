# Contract Template Catalog

This page summarizes SatoshiNet smart contract templates, runtimes, and testnet entries. For more detailed protocol rules, see [Smart Contract Protocol](../protocol/contracts/readme.md).

## Current Status Matrix

| Type | Status | Testnet Entry | Notes |
| --- | --- | --- | --- |
| Agent / Prediction Contract | Implemented / Testing | [Prediction Contract Test](../use/prediction-contract.md) | First public testnet validation scenario |
| Template Contract: AMM | Implemented / Testnet Iterating | PWA `Tools -> Smart Contracts`, [Deploy AMM Pool](amm-pool-quickstart.md) | Smart contract template test capability, not market AMM |
| Template Contract: LimitOrder | Implemented / Testnet Iterating | PWA `Tools -> Smart Contracts`, [Deploy Limit Order Module](limit-order-quickstart.md) | Smart contract template test capability, not market limit order |
| Template Contract: Asset Exchange | Implemented / Testnet Iterating | To be added | Fixed-rule asset exchange scenario |
| EVM Runtime | Implemented / Testnet Iterating | [EVM Developer Preview](evm-quickstart.md) | Reuses Solidity / EVM ecosystem; invocation uses ABI calldata, and asset settlement still follows SatoshiNet UTXO model |
| EVM Sample: ConstantProductAMM | Implemented / Testing | PWA `Tools -> Smart Contracts`, [EVM Sample Contracts](evm-sample-contracts.md) | Solidity AMM standard sample, not market AMM |
| EVM Sample: LimitOrderBook | Implemented / Testing | PWA `Tools -> Smart Contracts`, [EVM Sample Contracts](evm-sample-contracts.md) | Solidity limit order standard sample, not market limit order |

## Template Documents Should Include

Each template contract should eventually include:

1. Contract type and template name.
2. Deployment parameters.
3. Invocation interfaces.
4. Input asset rules.
5. Result TX output rules.
6. Permission boundary.
7. Fees and GAS.
8. Explorer / Indexer verification method.
9. Testnet contract address or example txid.
10. Known limitations.

## Current Priorities

1. Prediction contract: complete user testing, deployer quickstart, and testnet evidence.
2. AMM template contract: complete deployment, swap, add liquidity, remove liquidity, and verification flow.
3. LimitOrder template contract: complete order creation, filling, cancellation, and Result TX verification flow.
4. EVM sample contracts: add testnet addresses, txids, calldata generation records, and Explorer evidence for `ConstantProductAMM` and `LimitOrderBook`.
5. EVM Runtime: add RPC, Chain ID, example repo, Solidity deployment flow, estimate flow, and ABI calldata invocation flow.

**Page Status: Planning**
