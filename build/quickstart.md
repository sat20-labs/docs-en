# Developer Quickstart

This page gives developers the shortest path into the SAT20 / SatoshiNet ecosystem.

## Choose What You Are Building

| What you build | Starting point |
| --- | --- |
| STP wallet or client | `sat20-agent-wallet` skill and adapter contract |
| Indexer / data service | L1/L2 indexers, asset fact layer, and multi-protocol asset state |
| SatoshiNet application | Smart contract docs and testnet |
| Exchange integration | Indexers, STP state, deposits, and withdrawals |
| AI Agent | SAT20 Agent Wallet skill, PWA adapter, safety verification matrix |
| Explorer or data service | L1/L2 indexers and transaction state model |

## Install Wallet and Agent Skill

For ordinary users and mainnet scenarios, install and initialize SAT20 PWA Wallet before installing SAT20 Agent Wallet skill. The PWA wallet is the security boundary for private keys, mnemonics, signing, authorization prompts, and the channel database. The skill is the Agent's operation knowledge and workflow.

Install SAT20 PWA Wallet:

```text
https://sat20.org/pwa/?install=1
```

After creating or importing a wallet inside the PWA, backing it up, and unlocking it, install the skill:

```bash
curl -fsSL https://raw.githubusercontent.com/sat20-labs/docs/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/scripts/install.sh | bash
```

After installation, the Agent should call the wallet adapter through `SAT20_ADAPTER_URL` or `SAT20_CLIENT_CMD`.

The canonical skill source is maintained in `sat20-labs/docs`; it is not copied into `docs-en`.

## Implement an Adapter

A minimal adapter should support:

1. `wallet.status`
2. `stp.status`
3. `stp.safety_snapshot`
4. `stp.open`
5. `stp.splicing_in`
6. `stp.unlock`
7. `stp.lock`
8. `stp.splicing_out`
9. `stp.transaction`

Full contract: [adapter contract](https://github.com/sat20-labs/docs/blob/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/references/adapter-contract.md).

## Integrate Indexers

If you are building a wallet, exchange, explorer, or Agent, start with [Indexer Integration and Asset Fact Layer](indexer.md). A minimal implementation should query L1 UTXO assets, transaction confirmations, spent state, L2 UTXOs, ascend / descend events, and channel-related state.

Interfaces are still evolving quickly. Use the [API Source Map](api-source-map.md) to find authoritative source entries. The old Swagger only covers part of the early ORDX indexer and is not the complete current SAT20 / SatoshiNet API documentation.

## Testnet Acceptance

Developers should verify at least the following on testnet:

1. Open an ordinary client-core channel.
2. Splice in one protocol asset.
3. Run unlock / lock round trip.
4. Splice out back to Bitcoin L1.
5. Export commitment transactions.
6. Verify punish coverage.
7. Handle unknown network results.
8. Verify L1/L2 asset evidence through indexers.

Checklist: [Third-Party STP Client Implementation Checklist](../protocol/stp/implementation-checklist.md).
