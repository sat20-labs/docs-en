# Build: Developer Center

The Build section is for developers, wallets, exchanges, indexers, smart contract developers, and AI Agent builders.

The SatoshiNet ecosystem needs more than a protocol. It needs a complete developer system: wallets, SDKs, indexers, contract templates, testnets, documentation, examples, and community support. STP and indexers form the foundation for moving assets into SatoshiNet: STP protects control, while indexers provide asset facts.

## Developer Paths

| Goal | Path |
| --- | --- |
| Build an STP client | Read the STP client integration guide and implement a JSON adapter |
| Integrate a wallet | Use the PWA Wallet adapter or implement your own secure wallet boundary |
| Integrate an exchange | Focus on deposits, withdrawals, L1/L2 state, channel state, and indexers |
| Find API entry points | Use the API Source Map to locate L1 indexer, SatoshiNet RPC, L2 indexer, and wallet WASM/PWA adapter sources |
| Integrate indexers | Understand Bitcoin L1 and SatoshiNet L2 asset fact layers, multi-protocol assets, confirmations, reorg, and cross-layer evidence |
| Build contract applications | Start with template contracts and the GAS model |
| Build AI Agents | Install SAT20 Agent Wallet skill and implement safety verification plus authorization flows |
| Run infrastructure | Understand Core Nodes, indexers, explorers, and the testnet |

## Start

1. Read the [Developer Quickstart](quickstart.md).
2. Read the [API Source Map](api-source-map.md) to find current authoritative source entries.
3. Read [Indexer Integration and Asset Fact Layer](indexer.md).
4. Read [Third-Party STP Client Integration Guide](../protocol/stp/client-integration.md).
5. Read [Third-Party STP Client Implementation Checklist](../protocol/stp/implementation-checklist.md).
6. If you build an exchange or wallet, read [Exchange and Wallet Integration](exchange-and-wallet.md).

## Principles

1. Core Node status is not final fact.
2. A single indexer response is not unquestionable final fact; critical operations need txid, vout, height, confirmations, and cross-source evidence.
3. Balance display is not an asset safety proof.
4. Mainnet operations preserve user authorization.
5. Stop value movement when commitment transactions or punish coverage are missing.
6. Unknown-result network errors are treated as "possibly already successful."
