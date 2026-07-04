# Developer Quickstart

This page gives the shortest path into SAT20 / SatoshiNet development.

## Choose a Runnable Target

| Target | Guide |
| --- | --- |
| Understand EVM Developer Preview | [EVM Developer Preview](evm-quickstart.md) |
| View EVM sample contracts | [EVM Sample Contracts](evm-sample-contracts.md) |
| Build a community DEX / DAO | [Community DEX / DAO Quickstart](community-dex-quickstart.md) |
| Deploy a community DAO | [DAO Quickstart](dao-quickstart.md) |
| Deploy a smart contract template AMM pool | [AMM Pool Quickstart](amm-pool-quickstart.md) |
| Deploy Launchpad | [Launchpad Quickstart](launchpad-quickstart.md) |
| Deploy a smart contract template limit order module | [Limit Order Quickstart](limit-order-quickstart.md) |
| Run Core Node, Indexer, Explorer | [Infrastructure Quickstart](infrastructure-quickstart.md) |
| Integrate Wallet SDK | [Wallet SDK Quickstart](wallet-sdk-quickstart.md) |
| Build a white-label DEX | [White-Label DEX](white-label-dex.md) |
| Build a SatoshiNet AI Agent | [AI Agent Quickstart](ai-agent-quickstart.md) |
| Check contract template status | [Contract Template Catalog](contract-template-catalog.md) |
| Integrate wallet or exchange | [Exchange and Wallet Integration](exchange-and-wallet.md) |

Pages with status labels are still planning or depend on systems that are in development.

## Testnet Acceptance

Developers should eventually verify at least:

1. Open a normal client-core channel.
2. Splice in one protocol asset.
3. Unlock / lock round trip.
4. Splice out back to Bitcoin L1.
5. Export commitment transaction.
6. Verify punish coverage.
7. Handle unknown network results.
8. Use indexers to verify L1/L2 asset evidence.
9. Deploy or invoke at least one testnet smart contract, such as Prediction, or test template AMM, template limit order, EVM `ConstantProductAMM`, or EVM `LimitOrderBook` from PWA `Tools -> Smart Contracts`.

See [Third-Party STP Client Implementation Checklist](../protocol/stp/implementation-checklist.md).

**Page Status: Planning**
