# SAT20 / SatoshiNet

## SatoshiNet Is an Open Bitcoin-Native Execution Network

SatoshiNet lets BTC communities build, own, and operate their own financial infrastructure. Bitcoin L1 provides asset origin, final settlement, and dispute boundaries. Indexers provide queryable and verifiable asset facts. STP provides cross-layer asset control, exits, and punishment paths for revoked states. SatoshiNet provides transactions, smart contracts, and application execution. Wallets and AI Agents provide user authorization, safety checks, and interaction.

SAT20 is the protocol and open-source technology stack around Bitcoin-native assets. SatoshiNet is the open execution network in the SAT20 stack.

- [Build a DEX / DAO for My Community](community-stack/readme.md)
- [View Current Capabilities and Evidence](ecosystem/satoshinet-today.md)
- [Run Nodes and Infrastructure](run/readme.md)
- [Understand GAS and Node Economics](network-economics/readme.md)
- [Join the Builder Program](ecosystem/builder-program.md)

## Names and Relationships

| Name | Positioning |
| --- | --- |
| SAT20 | Protocol and open-source technology stack around Bitcoin-native assets |
| SatoshiNet | Open Bitcoin-native execution network |
| Indexer | Asset fact layer for Bitcoin L1 and SatoshiNet |
| STP / Transcend | Cross-layer asset control protocol between Bitcoin L1 and SatoshiNet |
| ORDX | Satoshi-denominated asset protocol in the SAT20 stack |
| GAS | SatoshiNet-native network fee and security staking asset |

## Why SatoshiNet Exists

Bitcoin L1 is well suited for scarce assets, final settlement, and long-term security. It is not designed to carry every high-frequency transaction, complex contract, community governance flow, or application execution. Scaling execution is a natural direction for Bitcoin, but the hard problem is preserving links to Bitcoin asset facts, user control, and exit paths while gaining better execution.

SatoshiNet explores an open path:

1. Assets originate from Bitcoin L1.
2. Asset facts can be traced and verified.
3. Users retain protection and exit paths in abnormal situations.
4. Applications execute in a faster, lower-cost, programmable network.
5. Communities and third parties can run infrastructure independently instead of permanently depending on SAT20 Labs.

## From Bitcoin Facts to Programmable Execution

```text
Bitcoin L1
   ↓
Indexer: Asset Facts
   ↓
STP: Control and Exit
   ↓
SatoshiNet: Transactions and Contract Execution
   ↓
DEX · DAO · Wallet · Explorer · AI Agent
```

## What You Can Do Now

| Capability | Use | Entry |
| --- | --- | --- |
| Community infrastructure | Plan nodes, indexers, explorers, wallets, DEX, DAO, and operations backend for BTC communities | [Community Stack](community-stack/readme.md) |
| Bring assets into SatoshiNet | Use Indexer to identify Bitcoin L1 asset facts and STP to place assets into a user-exitable channel safety boundary | [STP Introduction](learn/stp.md) |
| User flows | Install wallet, enter SatoshiNet, complete first swap, and verify with Explorer | [Use](use/readme.md) |
| Developer integration | Integrate Indexer, wallet, STP, contracts, and community DEX / DAO | [Build](build/readme.md) |
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

Users and Agents need to verify:

1. Whether assets are on Bitcoin L1, in an STP channel, in a SatoshiNet user address, or in a contract address.
2. Whether critical transactions can be traced to txid, vout, height, and confirmations.
3. Whether the wallet holds the latest commitment transaction and required backup material.
4. Whether revoked states have punishment coverage.
5. Whether users still have exit or protection paths when a Core Node is offline, refuses service, or behaves maliciously.

## Status and Evidence

Docs use consistent status labels:

| Status | Meaning |
| --- | --- |
| Implemented | Code and core flow exist, but this does not automatically imply full production maturity |
| Testnet | Reproducible testnet flow exists |
| In Development | Code is being developed; full availability is not promised |
| Design in Progress | Rules, parameters, or governance are not finalized |
| Planned | Direction is defined but no verifiable implementation exists yet |
| Experimental | Research feature that may change or be removed |

Every status should eventually link to code, documentation, demo, Explorer, contract address, test transaction, or validation record.

## Official Links

- Documentation: [docs.sat20.org](https://docs.sat20.org)
- Website: [sat20.org](https://sat20.org)
- X: [SAT20Labs](https://x.com/SAT20Labs)
- GitHub: [sat20-labs](https://github.com/sat20-labs)

This documentation serves users, developers, communities, node operators, wallets, exchanges, AI Agents, infrastructure teams, and strategic partners. The website explains outcomes, opportunities, and action paths. Docs provide protocol facts, implementation details, evidence, and risk boundaries.
