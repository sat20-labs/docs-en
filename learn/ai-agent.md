# AI Agents and User Asset Control

AI Agents are not an accessory to SAT20. They may become an important interface for ordinary users interacting with STP, SatoshiNet, and smart contracts.

STP, RSMC, commitment transactions, punishment transactions, indexer evidence, and cross-layer asset state are complex for most users. The value of an Agent is that it can read this evidence and translate complex protocols into executable, explainable, and verifiable user operations.

## Agent Responsibilities

An Agent is responsible for:

1. Understanding the user's goal.
2. Querying wallet, channel, L1/L2 transactions, and indexers.
3. Reviewing asset facts, confirmations, spent state, and cross-layer evidence.
4. Judging the current safety state.
5. Calling a wallet adapter.
6. Explaining what each step does.
7. Stopping when the state is unsafe.

Agent boundaries:

1. It does not store user mnemonic phrases.
2. It does not bypass wallet authorization.
3. It stops value movement when punish coverage cannot be proven.
4. It does not treat Core Node verbal status as a safety proof.

## Skill Model

SAT20 provides the SAT20 Agent Wallet skill so Agents can operate wallets and Core Nodes through a unified JSON adapter.

Recommended order:

1. The user first installs [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. The user creates or imports a wallet inside the PWA, completes backup, unlocks it, and selects the network.
3. The Agent then installs SAT20 Agent Wallet skill and calls the wallet through the PWA adapter.
4. The Agent first runs `wallet.status`, `stp.status`, and `stp.safety_snapshot` before moving assets.

Install the skill:

```bash
curl -fsSL https://raw.githubusercontent.com/sat20-labs/docs/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/scripts/install.sh | bash
```

The canonical skill source lives in the Chinese `sat20-labs/docs` repository. English documentation links to that source instead of copying a separate skill tree.

More details: [SAT20 Agent Wallet: Install and Use](../ai/sat20-agent-wallet/).
