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
| SatoshiNet Core Node | Implemented / Iterating | Mainnet, Public Testnet | Partial | See [Run the Network](../run/); third-party admission and operations docs remain incomplete |
| L1 Indexer | Implemented | Mainnet, testnet4 | Source map | See [Indexer Integration](../build/indexer.md) |
| L2 Indexer | Implemented | Mainnet, Public Testnet | Source map | See [API Source Map](../build/api-source-map.md) |
| Explorer | Implemented / Iterating | Mainnet, Testnet | User guide | See [Explorer Verification](../use/explorer-verification.md); some protocol details still require Indexer APIs |
| SAT20 PWA Wallet | Implemented / Iterating | Mainnet, Testnet | User guide / Source | [Open PWA Wallet](https://sat20.org/pwa/) |
| Wallet SDK | Implemented / Iterating | Mainnet, Testnet | Integration boundary | See [Wallet SDK Integration](../build/wallet-sdk-quickstart.md) |
| STP / Transcend | Implemented / Iterating | Mainnet, Public Testnet | Protocol spec / Test evidence | See [STP Technical Whitepaper](../protocol/stp/) |
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
| DKVS | Implemented / Testnet Iterating | Public Testnet | Protocol spec / E2E | Signed records, CAS, subscriptions, FREE_LOCAL, and AUTOPAY sync are implemented; see [DKVS](../protocol/dkvs/readme.md) |
| RGB11 L1 wallet loop | Implemented / Iterating | Bitcoin L1 test flows | Protocol spec / Wallet tests | Issue, import, invoice, send, and recovery are integrated; STP cross-layer support is not open, see [RGB11](../protocol/rgb11/readme.md) |
| D-Indexer | R&D | Not deployed as a standalone public product | Design notes | Kept separate from implemented DKVS |
| Community Builder Agent | Planned / Experimental | Not Deployed       | Missing       | See [Community Builder Agent](../ai/community-builder-agent.md)        |
| Mining Node | Implemented / Admission Rules in Design | Mainnet, Testnet | Partial | Node software runs; open admission, staking, and penalties remain in design, see [Mining Node](../run/mining-node.md) |
| Core Node | Implemented / Admission Rules in Design | Mainnet, Testnet | Partial | Node and STP services run; third-party admission remains in design, see [Core Node](../run/core-node.md) |
| Gas metering / test SGAS | Implemented / Economics in Design | Running Networks, Public Testnet | Draft | Transaction, contract, and AUTOPAY fee paths are implemented; formal GAS issuance, staking, distribution, and governance remain in design |

## Evidence to Add for Each Capability

1. GitHub repo / commit / release.
2. Demo or testnet entry.
3. Contract address or txid.
4. Explorer / Indexer evidence.
5. Test records.
6. Known limitations.
7. Last verification date.

**Last checked against source and public entries: 2026-08-29. Page Status: Maintained**
