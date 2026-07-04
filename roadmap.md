# Roadmap

The SAT20 / SatoshiNet roadmap is organized around one long-term goal: building an open Bitcoin-native execution network where Bitcoin L1 assets can enter a programmable, low-cost, automatable application environment while preserving user control.

The roadmap is not a price promise and not a fixed-date schedule. Every completed item should map to public code, documentation, transactions, contract addresses, test records, or external deployment cases whenever possible.

## Now

| Item                                       | Status             | Acceptance Criteria                                                                         | Contribution Entry                                                        |
| ------------------------------------------ | ------------------ | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Smart contract public testnet              | Testing            | PWA can claim test GAS, deploy or invoke contracts, and verify transactions and Result TX through Explorer / Indexer | [Prediction Contract Test](use/prediction-contract.md) |
| Prediction / Agent contract testing        | Testing            | Users can bet in Market, deployers can deploy matches from Tools, and result confirmation and settlement paths are reviewable | [Prediction Contract Test](use/prediction-contract.md) |
| Template AMM / limit order smart contract testing | Testing | Deploy and invoke from PWA `Tools -> Smart Contracts`, and verify asset settlement through Result TX; do not confuse with market AMM / limit order Channel Contracts | [Contract Template Catalog](build/contract-template-catalog.md) |
| EVM sample contract testing                | Testing            | Verify `ConstantProductAMM` and `LimitOrderBook` samples from PWA `Tools -> Smart Contracts`, plus Solidity apps and SatoshiNet asset interfaces | [EVM Sample Contracts](build/evm-sample-contracts.md) |
| Real Community Stack deployment docs       | Planning           | At least one reproducible community DEX / DAO testnet flow                                  | [Community Stack](community-stack/)                                       |
| First DEX / DAO pilots                     | Planning           | Testnet entry, contract or transaction evidence, and user guide                             | [Builder Program](ecosystem/builder-program.md)                           |
| EVM Developer Preview                      | Testnet Iterating  | RPC, Chain ID, Faucet, sample repository, and Explorer verification are defined             | [EVM Developer Preview](build/evm-quickstart.md)                          |
| Testnet user loop                          | Planning           | Wallet, test assets, first swap, Explorer verification, exit, and recovery are reproducible | [Use](use/)                                                               |
| Mining / Core Node and GAS economics draft | Design in Progress | Staking, fees, penalties, exit, and open questions are public                               | [Network Economics](network-economics/)                                   |
| Builder Program application entry          | Planning           | A submittable form or GitHub Issue Template exists                                          | [Builder Program](ecosystem/builder-program.md)                           |
| Sustainable protocol development support   | Planning           | Support methods, fund usage, deliverables, and responsible entity are clear                 | [Support Protocol Development](governance-support/support-development.md) |

## Next

| Item                                 | Status         | Acceptance Criteria                                                                                | Contribution Entry                                       |
| ------------------------------------ | -------------- | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| Reproducible white-label DEX toolkit | Planning       | Frontend, backend, contracts, Indexer, Explorer, and wallet integration can be deployed            | [White-Label DEX](build/white-label-dex.md)              |
| Third-party nodes and indexers       | Planning       | External teams can run them and provide public evidence                                            | [Run the Network](run/)                                  |
| EVM SDK / RPC                        | Testnet Iterating | Minimal contract deploy, invoke, event, and Result TX are verifiable                               | [EVM Contracts](protocol/contracts/evm.md)               |
| Community Builder Agent              | In Development | Requirement intake, config draft, human confirmation, testnet deployment plan, and evidence report | [Community Builder Agent](ai/community-builder-agent.md) |
| First external ecosystem cases       | Planning       | Built on SatoshiNet includes at least one external project                                         | [Built on SatoshiNet](ecosystem/built-on-satoshinet.md)  |
| Core English docs                    | Planning       | Homepage, Community Stack, Today, Security, Nodes, GAS, and Builder Program are synced in English  | docs-en                                                  |

## Later

| Item                                         | Status             | Acceptance Criteria                                                                        | Contribution Entry |
| -------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------ | ------------------ |
| Conversational semi-automated deployment     | Planned            | Agent can generate deployment config and execute after administrator confirmation          |                    |
| Multi-community shared liquidity             | Planned            | AMM, limit order, aggregation, and cross-community routing are verifiable                  |                    |
| More open node admission                     | Design in Progress | Node registration, staking, exit, penalties, and public status page are defined            |                    |
| Future foundation and public Treasury Policy | Planned            | Legal structure, governance, multisig, funding sources, and conflict policy are public     |                    |
| SIP protocol improvement process             | Planned            | Proposal format, discussion process, versioning, and governance boundary are defined       |                    |
| More complete developer support system       | Planned            | Grant, sponsorship, service contract, audit, and docs collaboration mechanisms are defined |                    |

## Principles

1. Separate plans from reality.
2. Express safety with evidence, not absolute slogans.
3. Do not use price, fixed-yield, or buyback narratives for GAS or node economics.
4. AI Agents are an interface and automation layer for complex infrastructure, not the reason SatoshiNet exists.
5. The goal of an open network is to let communities, developers, nodes, and infrastructure teams participate independently.
