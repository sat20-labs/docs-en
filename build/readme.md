# Developer Center

Build is for developers, wallets, exchanges, indexers, smart contract developers, and AI Agent builders.

SatoshiNet needs a complete developer system: wallets, SDK, indexers, contract templates, testnet, docs, examples, and community support. STP and Indexer together form the foundation for bringing assets into SatoshiNet: STP protects control, and Indexer provides asset facts.

## Quick Starts

| First Thing You Want to Do | Entry |
| --- | --- |
| Understand EVM Developer Preview | [EVM Developer Preview](evm-quickstart.md) |
| View EVM sample contracts | [EVM Sample Contracts](evm-sample-contracts.md) |
| Build a community DEX / DAO | [Community DEX / DAO Quickstart](community-dex-quickstart.md) |
| Deploy a community DAO | [DAO Quickstart](dao-quickstart.md) |
| Deploy a smart contract template AMM pool | [AMM Pool Quickstart](amm-pool-quickstart.md) |
| Deploy Launchpad | [Launchpad Quickstart](launchpad-quickstart.md) |
| Deploy a smart contract template limit order module | [Limit Order Quickstart](limit-order-quickstart.md) |
| Run Core Node, Indexer, or Explorer | [Infrastructure Quickstart](infrastructure-quickstart.md) |
| Integrate Wallet SDK | [Wallet SDK Quickstart](wallet-sdk-quickstart.md) |
| Build a white-label DEX | [White-Label DEX](white-label-dex.md) |
| Build a SatoshiNet AI Agent | [AI Agent Quickstart](ai-agent-quickstart.md) |
| Check contract template status | [Contract Template Catalog](contract-template-catalog.md) |
| Integrate wallet or exchange | [Exchange and Wallet Integration](exchange-and-wallet.md) |
| Locate authoritative API source | [API Source Map](api-source-map.md) |

Pages with status labels are still planning or depend on systems that are in development. They are kept as entries so they can be completed over time.

Independent quickstarts should later include environment requirements, testnet addresses, example code, expected output, Explorer verification, and common errors.

## Developer Paths

| Goal | Path |
| --- | --- |
| Build an STP client | Read the STP client integration guide, implement a JSON adapter, and verify with the testnet acceptance checklist |
| Integrate a wallet | Use the PWA Wallet adapter or implement your own secure wallet boundary |
| Integrate an exchange | Focus on deposits, withdrawals, L1/L2 state, channel state, and indexers |
| Locate API entries | Use the API Source Map to locate L1 Indexer, SatoshiNet RPC, L2 Indexer, and wallet WASM/PWA adapter |
| Integrate Indexer | Understand the asset fact layer across Bitcoin L1 and SatoshiNet L2, including multi-protocol assets, confirmations, reorgs, and cross-layer evidence |
| Build contract apps | Start from template contracts and the GAS model |
| Build AI Agents | First connect to the PWA wallet safety boundary, then install the SAT20 Agent Wallet skill and implement safety verification and authorization flows |
| Run infrastructure | Understand Core Node, Indexer, Explorer, and testnet |

## Start

1. Read [Developer Quickstart](quickstart.md).
2. Choose one runnable quickstart and complete one real testnet operation.
3. Read [API Source Map](api-source-map.md) to confirm authoritative source entries.
4. Read [Indexer Integration and Asset Fact Layer](indexer.md).
5. Read [Third-Party STP Client Integration Guide](../protocol/stp/client-integration.md).
6. Read [Third-Party STP Client Implementation Checklist](../protocol/stp/implementation-checklist.md).
7. If you are building an exchange or wallet, read [Exchange and Wallet Integration](exchange-and-wallet.md).

## Principles

1. Core Node state is not final truth.
2. A single indexer response is not unquestionable final truth; critical operations require txid, vout, height, confirmations, and cross-source evidence.
3. Balance display is not asset safety proof.
4. Mainnet operations require user authorization.
5. Stop value movement if commitment transaction or punishment coverage is missing.
6. Treat unknown network results as "possibly already successful."
