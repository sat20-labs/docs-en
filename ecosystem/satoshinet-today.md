# SatoshiNet Today: Current Capabilities

This page shows capabilities that SatoshiNet already has or is building. It is not a marketing list; it is a public status index.

## Status Dimensions

| Dimension              | Meaning                                             |
| ---------------------- | --------------------------------------------------- |
| Implementation Status  | Implemented, In Development, Planned, Experimental  |
| Available Environment  | Public Testnet, Limited Test, Mainnet, Not Deployed |
| Documentation Evidence | Complete, Partial, Missing, Protocol Spec           |

## Capability Matrix

| Module                  | Implementation         | Environment        | Docs Evidence | Evidence / Next Step                                                   |
| ----------------------- | ---------------------- | ------------------ | ------------- | ---------------------------------------------------------------------- |
| SatoshiNet Core Node    | Implemented            | Public Testnet     | Partial       | See [Run the Network](../run/)                                         |
| L1 Indexer              | Implemented            | Available          | Partial       | See [Indexer Integration](../build/indexer.md)                         |
| L2 Indexer              | Implemented            | Public Testnet     | Partial       | See [API Source Map](../build/api-source-map.md)                       |
| Explorer                | Implemented            | Testnet            | Partial       | Add unified entry and verification cases                               |
| SAT20 PWA Wallet        | Implemented            | Testnet            | Partial       | [Install PWA Wallet](https://sat20.org/pwa/?install=1)                 |
| Wallet SDK              | Implemented            | Available          | Partial       | See [Exchange and Wallet Integration](../build/exchange-and-wallet.md) |
| STP / Transcend         | Implemented            | Public Testnet     | Partial       | See [STP Technical Whitepaper](../protocol/stp/)                       |
| L2 market AMM Channel Contract | Implemented | Testnet | User guide | See [Provide AMM Liquidity](../use/amm-liquidity.md) |
| L2 market limit order Channel Contract | Implemented | Testnet | User guide | See [Use Limit Orders](../use/limit-order.md) |
| Launchpad               | Implemented            | Testnet            | Partial       | Add manual and case                                                    |
| DAO / Community Fund    | Implemented            | Limited Testnet    | Partial       | Add templates, UID, donations, airdrops, governance flow               |
| Smart contract framework | Implemented | Public Testnet | Partial | See [Smart Contract Protocol](../protocol/contracts/readme.md) |
| EVM Runtime | Implemented / Iterating | Public Testnet | Protocol Spec | See [EVM Contracts](../protocol/contracts/evm.md) |
| Template AMM smart contract | Implemented / Testing | Public Testnet | Deployment guide | Only through PWA `Tools -> Smart Contracts`; see [Deploy AMM Pool](../build/amm-pool-quickstart.md) |
| Template limit order smart contract | Implemented / Testing | Public Testnet | Deployment guide | Only through PWA `Tools -> Smart Contracts`; see [Deploy Limit Order Module](../build/limit-order-quickstart.md) |
| EVM ConstantProductAMM sample | Implemented / Testing | Public Testnet | Partial | See [EVM Sample Contracts](../build/evm-sample-contracts.md) |
| EVM LimitOrderBook sample | Implemented / Testing | Public Testnet | Partial | See [EVM Sample Contracts](../build/evm-sample-contracts.md) |
| Agent / Prediction Contract | Implemented / Testing | Public Testnet | User guide / Protocol Spec | See [Prediction Contract Test](../use/prediction-contract.md) and [Natural Language Contracts](../protocol/contracts/agent.md) |
| Community Builder Agent | Planned / Experimental | Not Deployed       | Missing       | See [Community Builder Agent](../ai/community-builder-agent.md)        |
| Mining Node | In Development | Testnet | Missing | See [Mining Node](../run/mining-node.md) |
| Core Node | In Development | Testnet | Missing | See [Core Node](../run/core-node.md) |
| GAS Economics           | Design in Progress     | Not Deployed       | Draft         | See [Network Economics](../network-economics/)                         |

## Evidence to Add for Each Capability

1. GitHub repo / commit / release.
2. Demo or testnet entry.
3. Contract address or txid.
4. Explorer / Indexer evidence.
5. Test records.
6. Known limitations.
7. Last verification date.

**Page Status: In Development**
