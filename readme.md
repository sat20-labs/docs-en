# SAT20 / SatoshiNet

## SatoshiNet Is the Native Execution Layer for Bitcoin’s Future Value Network

SAT20 is a protocol and open-source technology stack around Bitcoin-native assets. SatoshiNet is the open execution network in the SAT20 stack. It does not replace Bitcoin L1. It extends Bitcoin L1 asset facts, user control, and final settlement into a faster, lower-cost, programmable, and automatable execution environment.

Our long-term thesis is: **Bitcoin will become the foundation of the future value network, and SatoshiNet will become a core network for carrying services, applications, and AI Agents in that value network.**

“Helping BTC communities build, own, and operate their own financial infrastructure” remains an important goal, but it is more accurately a near-term adoption path rather than the final vision. Community DEX, DAO, wallet, Indexer, Explorer, nodes, and Launchpad are the first scenarios for validating SAT20’s native extension path, creating real usage, fee flow, and ecosystem coordination.

## SAT20 in One Paragraph

```text
Bitcoin L1 provides asset origin, UTXO facts, final settlement, and dispute boundaries.
Indexer provides queryable and reviewable asset facts.
STP provides cross-layer asset control, exits, and punishment paths for revoked states.
SatoshiNet provides transactions, smart contracts, applications, and service execution.
AI Agents, DKVS, and distributed L1 indexing serve the future open value network.
```

## Names and Relationships

| Name | Positioning |
| --- | --- |
| SAT20 | Protocol and open-source technology stack around Bitcoin-native assets |
| SatoshiNet | Native execution layer for Bitcoin’s future value network |
| Indexer | Mainnet asset fact layer for Bitcoin L1 and SatoshiNet |
| STP / Transcend | Mainnet cross-layer asset control, exit, and punishment protocol between Bitcoin L1 and SatoshiNet |
| SatoshiNet Core | Mainnet execution network for transactions, contracts, applications, and services |
| EVM Runtime | EVM execution environment implemented and deployed to testnet |
| DKVS | Distributed key-value infrastructure direction in development |
| Distributed L1 Indexer | Lightweight, distributed L1 asset-fact verification network in development |
| ORDX | Satoshi-denominated asset protocol in the SAT20 stack |
| GAS | SatoshiNet-native network fee and security staking asset |

## Why SatoshiNet Exists

Bitcoin L1 is well suited for scarce assets, final settlement, and long-term security. It is not designed to carry every high-frequency transaction, complex contract, community governance flow, AI Agent operation, or large-scale application execution. Scaling execution is a natural direction for Bitcoin, but the hard problem is preserving links to Bitcoin L1 asset facts, user control, and exit paths while gaining better execution.

SatoshiNet explores an open Bitcoin-native extension path:

1. Assets originate from Bitcoin L1.
2. Asset facts can be traced and verified.
3. Users retain protection and exit paths in abnormal situations.
4. Applications execute in a faster, lower-cost, programmable network.
5. Communities, developers, and third parties can run infrastructure independently instead of permanently depending on SAT20 Labs.
6. AI Agents can understand evidence, explain risk, generate plans, and execute actions without bypassing wallet authorization or holding user private keys.

## From Bitcoin Facts to Value-Network Services

```text
Bitcoin L1
   ↓
Indexer: Mainnet Asset Fact Layer
   ↓
STP: Mainnet Cross-Layer Control, Exit, and Punishment
   ↓
SatoshiNet Core: Mainnet Transaction, Contract, and Application Execution
   ↓
EVM Runtime: Testnet EVM Execution Environment
   ↓
DEX · DAO · Wallet · Explorer · Prediction · VSN · AI Agent
   ↓
DKVS · Distributed L1 Indexer · More Open Service Networks
```

## What You Can Do Now

| Capability | Use | Entry |
| --- | --- | --- |
| Understand SatoshiNet | Understand why Bitcoin needs native extension, the asset safety model, Indexer, STP, contracts, GAS, and AI Agents | [Learn: Understanding SatoshiNet](learn/readme.md) |
| Community infrastructure | Plan nodes, indexers, explorers, wallets, DEX, DAO, Launchpad, and operations backend for BTC communities | [Community Stack](community-stack/readme.md) |
| Bring assets into SatoshiNet | Use mainnet Indexer to identify Bitcoin L1 asset facts and mainnet STP to place assets into a user-exitable channel safety boundary | [STP Introduction](learn/stp.md) |
| User flows | Install wallet, enter SatoshiNet, complete first swap, and verify with Explorer | [Use SatoshiNet](use/readme.md) |
| Developer integration | Integrate Indexer, wallet, STP, SatoshiNet Core, contracts, and community DEX / DAO | [Developer Center](build/readme.md) |
| Network operation | Run Mining Node, Core Node, Indexer, Explorer, RPC, and monitoring | [Run the Network](run/readme.md) |
| Network economics | Understand GAS, fee flows, node staking, incentives, and open design questions | [Network Economics](network-economics/readme.md) |
| AI Agent operations | Let Agents perform asset safety checks and actions without holding private keys or bypassing wallet authorization | [AI Agent](ai/readme.md) |
| Ecosystem cooperation | Apply for pilots, contribute tools, provide liquidity, run nodes, or support protocol development | [Ecosystem](ecosystem/readme.md) |

## Choose Your Role

| Who You Are | Start Here |
| --- | --- |
| I run a BTC community | [Community Path](start-here/btc-community.md) |
| I am a Solidity / EVM developer | [Developer Path](start-here/developers.md) |
| I run infrastructure | [Infrastructure Path](start-here/infrastructure.md) |
| I am a wallet or exchange | [Wallet and Exchange Path](start-here/wallet-exchange.md) |
| I am an AI Agent developer | [AI Agent Path](start-here/ai-agent-builders.md) |
| I want to provide liquidity | [Liquidity Path](start-here/liquidity.md) |

## Security Is Expressed with Evidence

SatoshiNet does not use a centralized custodial bridge as its core cross-layer model. User protection depends on STP channel state, valid commitment transactions, punishment coverage, wallet backup, indexer evidence, and executable Bitcoin L1 paths.

Users, wallets, and Agents need to verify:

1. Whether assets are on Bitcoin L1, in an STP channel, in a SatoshiNet user address, or in a contract address.
2. Whether critical transactions can be traced to txid, vout, height, and confirmations.
3. Whether the wallet holds the latest commitment transaction and required backup material.
4. Whether revoked states have punishment coverage.
5. Whether users still have exit or protection paths when a Core Node is offline, refuses service, or behaves maliciously.
6. Whether design-stage capabilities are clearly marked rather than presented as finished products.

## Current Key Status

The website and docs use the same status language to express capability boundaries:

| Capability | Current Status | Notes |
| --- | --- | --- |
| Indexer | Implemented · Mainnet | Mainnet asset fact layer for Bitcoin L1 and SatoshiNet assets, transactions, confirmations, and protocol events |
| STP | Implemented · Mainnet | STP has been implemented and deployed to mainnet for cross-layer asset control, exit paths, and punishment coverage |
| SatoshiNet Core / Wallet / Explorer | Implemented · Mainnet | Core network, wallet entry, explorer, and base applications have mainnet foundations |
| EVM Runtime | Testnet | The EVM Runtime has been implemented and deployed to testnet for Solidity / ABI, UTXO asset-interface, and developer-experience validation |
| DKVS | In Development | Distributed key-value infrastructure is in development |
| Distributed L1 Indexer | In Development | Distributed L1 indexing is in development to reduce storage and compute requirements for indexer operators |
| VSN / Agent-Controlled Wallet / Long-Term Governance | Design / R&D | These areas remain under design, experimentation, and validation, and should not be presented as finished production capabilities |

## Status and Evidence

Docs use consistent status labels:

| Status | Meaning |
| --- | --- |
| Implemented | Code and core flow exist, but this does not automatically imply full production maturity |
| Mainnet | The capability is deployed to a mainnet environment, but production maturity should still be read together with documentation and risk boundaries |
| Testnet | Reproducible testnet flow exists |
| In Development | Code is being developed; full availability is not promised |
| Design in Progress | Rules, parameters, or governance are not finalized |
| Planned | Direction is defined but no verifiable implementation exists yet |
| R&D | Research infrastructure direction that may continue to change |
| Experimental | Research feature that may change or be removed |

Every status should eventually link to code, documentation, demo, Explorer, contract address, test transaction, or validation record.

## Official Links

- Website: [sat20.org](https://sat20.org)
- Documentation: [docs.sat20.org](https://docs.sat20.org)
- X: [SAT20Labs](https://x.com/SAT20Labs)
- GitHub: [sat20-labs](https://github.com/sat20-labs)

This documentation serves users, developers, communities, node operators, wallets, exchanges, AI Agents, infrastructure teams, and strategic partners. The website explains vision, outcomes, opportunities, and action paths. Docs provide protocol facts, implementation details, evidence, and risk boundaries.
