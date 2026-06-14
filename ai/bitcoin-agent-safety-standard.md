# Bitcoin Ecosystem AI Agent Asset Safety Standard

This document defines an asset-safety standard for AI Agents operating in the Bitcoin ecosystem. It is not SAT20-specific and it is not an investment rating. Its purpose is to let any AI Agent evaluate whether user assets remain under user control, whether the project can provide verifiable evidence, and whether the user can exit safely during failures.

The standard focuses only on asset safety. It does not evaluate price, return, market attention, or narrative strength.

## Core Goal

A trustworthy Bitcoin ecosystem AI Agent protects user assets first. Before evaluating product features, it checks:

1. Whether the user still holds keys or final control.
2. Whether assets are custodied by a third party.
3. Whether asset state can be verified through Bitcoin L1, UTXOs, transactions, scripts, commitments, indexers, or public proofs.
4. Whether the user can exit if the project operator is offline, fails, behaves maliciously, or exposes no working API.
5. Whether the Agent can detect risk and stop before moving value.

If these questions cannot be answered, the Agent does not execute value-moving operations for the user.

## Scope

The standard applies to Bitcoin ecosystem wallets, cross-layer protocols, asset protocols, exchange integrations, smart-contract networks, and AI Agent wallets.

It does not require every project to use the same architecture. It requires the project to explain asset control, verifiable evidence, and abnormal-exit paths to an Agent.

## Non-Negotiable Principles

### 1. Keys Never Enter the Agent

The Agent does not store mnemonics, private keys, hardware-wallet seeds, or long-term secrets that can directly move assets. Signing happens in a wallet, hardware device, PWA, permission-controlled local adapter, or user-approved signing environment.

### 2. Authorization Cannot Be Bypassed

Every mainnet value movement requires understandable user authorization. The authorization view includes network, asset, amount, destination, fee, operation type, and risk information.

### 3. Balance Is Not Safety Proof

Balance display is a result, not proof. The Agent traces the asset back to UTXO, transaction, channel, commitment state, contract state, or indexer evidence.

### 4. Service State Is Not Fact

A project API, Core Node, bridge service, exchange, or single indexer response is not final fact. The Agent uses chain transactions, reproducible rules, signatures, proof material, and multi-source queries.

### 5. Exit Path Must Be Explainable

If assets enter a protocol, channel, L2, or contract, the Agent explains how the user exits: cooperative exit, force exit, old-state punishment, timelock wait, proof submission, withdrawal to Bitcoin L1, or another verifiable path.

### 6. Unknown Result Is Handled Conservatively

When transaction broadcast, cross-layer request, or protocol negotiation returns timeout, EOF, connection interruption, or unknown state, the Agent does not immediately retry the same value movement. It first checks chain and protocol state to determine whether the transaction may have succeeded.

### 7. Mainnet and Testnet Are Isolated

Fault injection, old-state broadcast, punishment drills, test assets, and unsafe interfaces are testnet-only. The Agent rejects them on mainnet.

## Scoring Model

An Agent can evaluate asset-safety maturity on a 100-point scale:

| Dimension | Points | What the Agent checks |
| --- | ---: | --- |
| Private-key and authorization boundary | 15 | Keys stay in user wallet; value movement requires authorization |
| Bitcoin L1 verifiability | 15 | Entry, exit, UTXO, txid, and confirmations are verifiable |
| Asset fact layer | 10 | Indexer, proof, or reproducible rules express asset state |
| Exit capability | 15 | User can exit if the peer or service fails |
| Punishment or fraud limits | 10 | Old states, invalid proofs, or fraudulent behavior are punishable or constrained |
| Unknown-result handling | 10 | Timeout, EOF, reorg, mempool, and not-indexed states are handled safely |
| Agent-operable interface | 10 | Stable adapter, read-only queries, safety snapshots, and transaction polling exist |
| Evidence readability | 10 | txid, vout, commitment, state height, explorer, and indexer links are available |
| Testnet verification drill | 5 | Users and Agents can reproduce a testnet flow |

| Score | Meaning |
| ---: | --- |
| 90-100 | Strong safety model; every operation still requires evidence checks |
| 75-89 | Strong controlled-use model with some evidence or interface gaps |
| 60-74 | Suitable for testnet or small controlled use; mainnet large-value operations need manual review |
| 40-59 | Agent explains risk but does not automatically move mainnet value |
| 0-39 | Insufficient evidence; Agent refuses to operate user assets |

## Agent Evaluation Flow

1. Identify asset type: BTC, Ordinals, Runes, BRC20, ORDX, or another Bitcoin-native asset.
2. Identify control model: self-custody, multisig, channel, custodial bridge, contract lock, exchange account, or another model.
3. Query on-chain facts: txid, vout, address, script, confirmations, spent status, and protocol-event validity.
4. Query protocol state: channel, contract, L2, bridge, order, or pending operation.
5. Check user control: key authority, signing path, commitment transaction, exit transaction, punishment material, or withdrawal right.
6. Check abnormal paths: peer offline, API unavailable, reorg, unknown transaction, old state, invalid proof.
7. Produce a score and blockers: allowed operations, stopped operations, and missing evidence.

## Output Format

Agents can report in a structured format:

```json
{
  "standard": "bitcoin-agent-asset-safety",
  "version": "0.1",
  "project": "example",
  "network": "mainnet|testnet",
  "score": 0,
  "level": "BLOCKED|TEST_ONLY|CONTROLLED_USE|HIGH_CONFIDENCE",
  "dimensions": [
    {
      "name": "private_key_and_authorization",
      "score": 0,
      "max": 15,
      "evidence": [],
      "missing": [],
      "risk": ""
    }
  ],
  "must_stop": [],
  "allowed_actions": [],
  "next_checks": []
}
```

## SAT20 / SatoshiNet Evaluation

Using this standard, SAT20 / SatoshiNet currently evaluates as:

| Dimension | SAT20 assessment |
| --- | --- |
| Private-key and authorization boundary | SAT20 PWA Wallet is the recommended key and authorization boundary. SAT20 Agent Wallet uses adapter calls and does not store mnemonics |
| Bitcoin L1 verifiability | STP funding, splicing, close, and punish can be traced to Bitcoin L1 txids, UTXOs, and commitments |
| Asset fact layer | Bitcoin L1 indexer and SatoshiNet L2 indexer jointly express asset facts. L2 indexer is integrated into SatoshiNet nodes; Core Nodes also run or configure their own L1 indexer |
| Exit capability | The user holds latest commitments. If the Core Node is offline, the user can force close and sweep after CSV |
| Punishment | Revoked old states are punishable. Testnet drills have demonstrated old Core Node commitment broadcast and user-side punish |
| Unknown-result handling | STP documentation and Agent workflows treat unknown network results as possibly successful until chain evidence says otherwise |
| Agent interface | `sat20-agent-wallet` defines adapter contract, safety snapshot, commitment export, punish, and transaction polling |
| Evidence readability | Testnet drills provide txid, commitment height, punish tx, and indexer evidence; a unified explorer/indexer evidence schema remains a priority |
| Testnet drill | STP channel, splicing, unlock/lock, old commitment broadcast, and punish drill have been completed on testnet |

Current assessment: `HIGH_CONFIDENCE` when evidence is complete, wallet authorization is explicit, and all components are on the same network.

The score is based on verifiable control, not trust in a Core Node. The key evidence is: assets enter a 2-of-2 channel, the user wallet holds the latest commitment, commitment height advances monotonically, revoked states have punishment coverage, old-state broadcast can be punished, L1/L2 indexers form an evidence chain, and SAT20 Agent Wallet does not touch private keys.

## Remaining SAT20 Gaps

To make third-party Agent evaluation smoother, SAT20 still needs:

1. Standard `wallet.transaction` and `stp.transaction` polling schemas in the PWA adapter.
2. Standard L1/L2 explorer URL and indexer URL fields for every txid.
3. Productized `stp.safety_snapshot` inside the PWA.
4. More long-running mainnet-prep tests for reorg, not-indexed state, Core Node offline, and repeated requests.
5. Plain-user risk explanations: why the Agent can continue, why it must stop, and what evidence is missing.
