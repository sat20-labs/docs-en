# Developer Center

Build is for developers, wallets, exchanges, indexers, smart contract developers, and AI Agent builders.

SatoshiNet needs a complete developer system: wallets, SDK, indexers, contract templates, testnet, docs, examples, and community support. STP and Indexer together form the foundation for bringing assets into SatoshiNet: STP protects control, and Indexer provides asset facts.

## Quick Starts

| First Thing You Want to Do | Entry |
| --- | --- |
| Understand EVM Developer Preview | [EVM Developer Preview](evm-quickstart.md) |
| Build a community DEX / DAO | [Community DEX / DAO Quickstart](community-dex-quickstart.md) |
| Deploy a community DAO | [DAO Quickstart](dao-quickstart.md) |
| Deploy an AMM pool | [AMM Pool Quickstart](amm-pool-quickstart.md) |
| Deploy Launchpad | [Launchpad Quickstart](launchpad-quickstart.md) |
| Deploy limit order module | [Limit Order Quickstart](limit-order-quickstart.md) |
| Run Core Node, Indexer, or Explorer | [Infrastructure Quickstart](infrastructure-quickstart.md) |
| Integrate Wallet SDK | [Wallet SDK Quickstart](wallet-sdk-quickstart.md) |
| Build a white-label DEX | [White-Label DEX](white-label-dex.md) |
| Build a SatoshiNet AI Agent | [AI Agent Quickstart](ai-agent-quickstart.md) |
| Check contract template status | [Contract Template Catalog](contract-template-catalog.md) |
| Integrate wallet or exchange | [Exchange and Wallet Integration](exchange-and-wallet.md) |
| Locate authoritative API source | [API Source Map](api-source-map.md) |

Pages with status labels are still planning or depend on systems that are in development. They are kept as entries so they can be completed over time.

## Principles

1. Core Node state is not final truth.
2. A single indexer response is not unquestionable final truth; critical operations require txid, vout, height, confirmations, and cross-source evidence.
3. Balance display is not asset safety proof.
4. Mainnet operations require user authorization.
5. Stop value movement if commitment transaction or punishment coverage is missing.
6. Treat unknown network results as "possibly already successful."
