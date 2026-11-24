# SAT20 / SatoshiNet

## SatoshiNet Is the Native Execution Layer for Bitcoin’s Future Value Network

SAT20 is a protocol and open-source technology stack around Bitcoin-native assets. SatoshiNet is the open execution network in the SAT20 stack. It does not replace Bitcoin L1. It extends Bitcoin L1 asset facts, user control, and final settlement into a faster, lower-cost, programmable, and automatable execution environment.

Our long-term thesis is: **Bitcoin will become the foundation of the future value network, and SatoshiNet will become a core network for carrying services, applications, and AI Agents in that value network.**

“Helping BTC communities build, own, and operate their own financial infrastructure” remains an important goal, but it is more accurately a near-term adoption path rather than the final vision. Community DEX, DAO, wallet, Indexer, Explorer, nodes, and Launchpad are the first scenarios for validating SAT20’s native extension path, creating real usage, fee flow, and ecosystem coordination.

## Core Capability Chain and R&D Directions

```text
Indexer
   ↓
STP
   ↓
SatoshiNet
   ↓
Smart Contracts
   ↓
DKVS
   ↓
AI Agent Wallet
```

This chain shows the current capability relationship: Indexer provides asset facts; STP provides cross-layer control; SatoshiNet provides the execution network; Smart Contracts provide programmable logic; DKVS provides account and application small-data synchronization; and AI Agent Wallet evolves from SAT20 Wallet into an Agent-facing authorization entry. D-Indexer remains a later R&D direction rather than a delivered layer in this chain.

## SAT20 in One Paragraph

```text
Bitcoin L1 provides asset origin, UTXO facts, final settlement, and dispute boundaries.
Indexer provides queryable and reviewable asset facts.
STP provides cross-layer asset control, exits, and punishment paths for revoked states.
SatoshiNet provides transactions, assets, blocks, contracts, and application execution.
Smart Contracts, DKVS, and AI Agent Wallet move SatoshiNet toward a more open value-network service layer. D-Indexer continues as an R&D direction.
```

## Names and Relationships

| Name | Positioning |
| --- | --- |
| SAT20 | Protocol and open-source technology stack around Bitcoin-native assets |
| SatoshiNet | Native execution layer for Bitcoin’s future value network |
| Indexer | Mainnet asset fact layer for Bitcoin L1 and SatoshiNet |
| STP / Transcend | Mainnet cross-layer asset control, exit, and punishment protocol between Bitcoin L1 and SatoshiNet |
| SatoshiNet | The latest mainnet update is complete. SatoshiNet supports transactions, asset transfers, STP channels, and channel contracts |
| Smart Contracts | Programmability layer on SatoshiNet. Users can deploy and invoke contracts on the public testnet. The mainnet launch is on hold |
| DKVS | Storage and synchronization layer for small records. DKVS is complete and is used for wallet account management and RGB11 backups |
| D-Indexer | Distributed L1 Indexer direction still in R&D; DKVS is one reusable foundation, but the two are not one delivered product |
| SAT20 Wallet | Available as a browser extension and as the [PWA Wallet](https://sat20.org/pwa/), which is now live |
| AI Agent Wallet | Agent authorization and operation entry evolving from SAT20 Wallet |
| ORDX | Satoshi-denominated asset protocol in the SAT20 stack |
| GAS | Planned asset for SatoshiNet network fees and node staking. The mainnet asset name and total supply are still undecided |

## Why SatoshiNet Exists

Bitcoin L1 is well suited for scarce assets, final settlement, and long-term security. It is not designed to carry every high-frequency transaction, complex contract, community governance flow, AI Agent operation, or large-scale application execution. Scaling execution is a natural direction for Bitcoin, but the hard problem is preserving links to Bitcoin L1 asset facts, user control, and exit paths while gaining better execution.

SatoshiNet explores an open Bitcoin-native extension path:

1. Assets originate from Bitcoin L1.
2. Asset facts can be traced and verified.
3. Users retain protection and exit paths in abnormal situations.
4. Applications execute in a faster, lower-cost, programmable network.
5. Communities, developers, and third parties can run infrastructure independently instead of permanently depending on SAT20 Labs.
6. AI Agents can understand evidence, explain risk, generate plans, and execute actions without bypassing wallet authorization or holding user private keys.

## From Bitcoin Facts to Agent Wallet

```text
Bitcoin L1: Asset origin, UTXO facts, final settlement, and dispute boundary
   ↓
Indexer: Mainnet asset fact layer
   ↓
STP: Mainnet cross-layer control, exit, and punishment
   ↓
SatoshiNet: Mainnet transactions, assets, blocks, and service execution
   ↓
Smart Contracts: The public testnet is open for deploying or invoking template contracts, Prediction contracts, and EVM samples
   ↓
DKVS: signed small data, account recovery, mailbox, and application-state coordination
   ↓
AI Agent Wallet: authorization entry based on SAT20 Wallet
```

D-Indexer is intentionally not shown above as a delivered layer. It remains an R&D direction for distributed L1 asset facts.

## What You Can Do Now

| Capability | Use | Entry |
| --- | --- | --- |
| Understand SatoshiNet | Understand why Bitcoin needs native extension, the asset safety model, Indexer, STP, contracts, GAS, and AI Agents | [Learn: Understanding SatoshiNet](learn/readme.md) |
| Community infrastructure | Plan nodes, indexers, explorers, wallets, DEX, DAO, Launchpad, and operations backend for BTC communities | [Community Stack](community-stack/readme.md) |
| Bring assets into SatoshiNet | Use mainnet Indexer to identify Bitcoin L1 asset facts and mainnet STP to place assets into a user-exitable channel safety boundary | [STP Introduction](learn/stp.md) |
| User flows | The SAT20 PWA Wallet is live. Use the PWA Wallet or browser extension to access SatoshiNet, complete swaps, and verify transactions in Explorer | [Use SatoshiNet](use/readme.md) |
| Test smart contracts | Switch the PWA Wallet to the public testnet, then claim test GAS to deploy or invoke contracts | [Get Test GAS and Test Contracts](use/test-assets-and-gas.md) |
| Developer integration | Integrate Indexer, STP, SatoshiNet, Smart Contracts, Wallet SDK, and community DEX / DAO | [Developer Center](build/readme.md) |
| Network operation | Run Mining Node, Core Node, Indexer, Explorer, RPC, and monitoring | [Run the Network](run/readme.md) |
| Network economics | Understand GAS, fee flows, node staking, incentives, and open design questions | [Network Economics](network-economics/README.md) |
| AI Agent operations | Let Agents perform asset safety checks and actions without holding private keys or bypassing wallet authorization | [AI Agent](ai/readme.md) |
| Ecosystem cooperation | Apply for pilots, contribute tools, provide liquidity, run nodes, or support protocol development | [Ecosystem](ecosystem/README.md) |

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
| SatoshiNet | Implemented · Mainnet | The latest mainnet update is complete. Asset transfers, STP channels, and channel contracts remain available. See [Current Capabilities](ecosystem/satoshinet-today.md) |
| Smart Contracts / EVM Runtime / Agent Contracts | Public Testnet / Mainnet on Hold | Users can deploy and invoke contracts on testnet. Key parameters such as the mainnet GAS asset name and total supply are still undecided, and the SatoshiNet Foundation has yet to be established. See [Smart Contract Protocol](protocol/contracts/readme.md) |
| DKVS | Complete | Used for wallet account management and RGB11 backups. See [DKVS](protocol/dkvs/readme.md) |
| D-Indexer | R&D / In Development | Distributed L1 asset-fact networking remains under development; DKVS implementation does not imply that D-Indexer is live |
| SAT20 Wallet / AI Agent Wallet | PWA Live / Agent Wallet Iterating | The PWA Wallet supports mainnet and testnet. See [Wallet and Assets](use/wallet-and-assets.md) |
| VSN / Long-Term Governance | Design / R&D | These areas remain under design, experimentation, and validation, and are not presented as finished production capabilities |

## Status and Evidence

Docs use consistent status labels:

| Status | Meaning |
| --- | --- |
| Implemented | Code and core flow exist, but this does not automatically imply full production maturity |
| Mainnet | The capability is deployed to a mainnet environment. Production maturity is read together with documentation and risk boundaries |
| Testnet | Reproducible testnet flow exists |
| In Development | Code is being developed; full availability is not promised |
| Iterating | The capability exists, but product experience, interfaces, or safety boundaries continue to improve |
| Design in Progress | Rules, parameters, or governance are not finalized |
| R&D | Research infrastructure direction that may continue to change |
| Experimental | Research feature that may change or be removed |

Every status links to code, documentation, demo, Explorer, contract address, test transaction, or validation record as evidence becomes available.

## Official Links

- Website: [sat20.org](https://sat20.org)
- Documentation: [docs.sat20.org](https://docs.sat20.org)
- X: [SAT20Labs](https://x.com/SAT20Labs)
- GitHub: [sat20-labs](https://github.com/sat20-labs)

This documentation serves users, developers, communities, node operators, wallets, exchanges, AI Agents, infrastructure teams, and strategic partners. The website explains vision, outcomes, opportunities, and action paths. Docs provide protocol facts, implementation details, evidence, and risk boundaries.
