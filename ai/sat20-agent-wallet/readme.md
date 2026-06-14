# SAT20 Agent Wallet

SAT20 Agent Wallet is a general skill and adapter specification that lets AI Agents operate SAT20 Wallet, STP channels, Bitcoin L1 assets, and SatoshiNet assets through a user-controlled wallet boundary.

Codex can install the skill directly. Other Agents can read the same `SKILL.md`, references, and scripts from the SAT20 docs repository.

## Design Goal

SAT20 Agent Wallet is built on the [Bitcoin Ecosystem AI Agent Asset Safety Standard](../bitcoin-agent-safety-standard.md). Before operating SAT20/STP, an Agent evaluates asset control, exit ability, chain evidence, Core Node risk, and missing data.

The skill does not put private keys into the Agent and does not hard-code one wallet implementation. It has three layers:

1. Skill: Agent workflows, safety guardrails, playbooks, and adapter contract.
2. Adapter scripts: JSON request forwarding to SAT20 PWA Wallet Adapter or another wallet client.
3. Wallet adapter: key management, transaction construction, signing, wallet/channel persistence, indexer access, SatoshiNet access, and Core Node communication.

The recommended user product is SAT20 PWA Wallet. The PWA loads `sat20wallet.wasm` and `stpd.wasm`, controls permissions, and exposes an adapter that the Agent can call. Keys, mnemonics, wallet database, signing, and user authorization stay inside the PWA.

## Canonical Skill Location

The SAT20 Agent Wallet skill is maintained only in the Chinese `sat20-labs/docs` repository:

```text
docs/ai/sat20-agent-wallet/skills/sat20-agent-wallet/
```

English documentation links to that canonical directory instead of carrying a copied skill tree.

Canonical GitHub directory:

```text
https://github.com/sat20-labs/docs/tree/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet
```

## Install

Official documentation entry: [docs.sat20.org](https://docs.sat20.org).

One-command install:

```bash
curl -fsSL https://raw.githubusercontent.com/sat20-labs/docs/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/scripts/install.sh | bash
```

By default the script installs to `~/.codex/skills/sat20-agent-wallet`. For a different Agent skill directory:

```bash
curl -fsSL https://raw.githubusercontent.com/sat20-labs/docs/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/scripts/install.sh | SAT20_SKILLS_DIR=/path/to/agent/skills bash
```

To pin a branch:

```bash
curl -fsSL https://raw.githubusercontent.com/sat20-labs/docs/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/scripts/install.sh | SAT20_DOCS_BRANCH=main bash
```

Codex can invoke it as `$sat20-agent-wallet` after installation. Other Agents that support `SKILL.md` can read:

```text
https://github.com/sat20-labs/docs/blob/main/ai/sat20-agent-wallet/skills/sat20-agent-wallet/SKILL.md
```

## Adapter Configuration

The preferred adapter is SAT20 PWA Wallet Adapter:

```bash
export SAT20_ADAPTER_URL="http://127.0.0.1:19530/stp-adapter"
```

The PWA adapter internally calls wallet/STP WASM modules, manages user authorization, and returns JSON evidence to the Agent.

For CLI-based adapters:

```bash
export SAT20_CLIENT_CMD="stp-client --json"
python3 "$SAT20_AGENT_WALLET_DIR/scripts/stp_adapter.py" '{"op":"stp.status","chain":"testnet"}'
```

The adapter accepts JSON requests and returns JSON responses. The skill is language-agnostic; adapters can be implemented in Go, JavaScript, Python, Rust, or other languages.

## Adapter Responsibilities

The adapter performs the real wallet and protocol work:

1. Manage keys or connect to a secure wallet.
2. Communicate with Core Nodes.
3. Build, sign, verify, and broadcast Bitcoin L1, SatoshiNet, and STP transactions.
4. Persist channel state, commitments, revocation material, and pending operations.
5. Return structured JSON to the Agent.

The skill defines workflows and safety gates. It does not custody keys.

## Supported Operations

### Wallet Management

| Operation | Purpose |
| --- | --- |
| `wallet.create` | Create a testnet wallet or ask PWA to create one |
| `wallet.import` | Import mnemonic into PWA wallet |
| `wallet.export_mnemonic` | Export mnemonic only after PWA authorization |
| `wallet.change_password` | Change wallet password after PWA authorization |
| `wallet.status` | Query wallet, network, WASM, and asset state |
| `wallet.send_assets` | Send Bitcoin L1 or SatoshiNet assets directly |
| `wallet.transaction` | Track wallet transaction state |

### Channel Management

| Operation | Purpose |
| --- | --- |
| `stp.status` | Query Core Node, channel, commitment height, and pending operations |
| `stp.open` | Open a private channel |
| `stp.reopen` | Reopen a closed client-Core channel when ledger evidence supports it |
| `stp.rebuild` | Rebuild channel state from L1/L2 ledger evidence without duplicate anchor |
| `stp.restore` | Restore state from peer, backup, or local persistence |
| `stp.expand` / `stp.expand_all` | Bring existing channel-address assets under commitment management |
| `stp.unlock` | Release channel assets to the user's SatoshiNet address |
| `stp.lock` | Lock SatoshiNet personal assets back into the channel |
| `stp.lock_with_expand` | Restore channel control when capacity is insufficient |
| `stp.splicing_in` | Bring Bitcoin L1 assets into the channel |
| `stp.splicing_out` | Exit channel assets to Bitcoin L1 |
| `stp.close` | Cooperative or force close |
| `stp.transaction` | Track STP operation state |

### Asset Safety

| Operation | Purpose |
| --- | --- |
| `stp.safety_snapshot` | Read channel point, commitment height, balances, CSV, and punishment coverage |
| `stp.commitment_export` | Export read-only commitment verification data |
| `stp.punish_status` | Query revoked-state punishment coverage |
| `stp.punish_build` | Build and dry-run a punishment transaction |
| `stp.punish_broadcast` | Broadcast a verified punishment transaction |
| `stp.force_close_plan` | Produce a unilateral exit plan |
| `stp.sweep_build` | Build, sign, verify, and optionally broadcast a CSV sweep |
| `stp.test_retain_server_commitment` | Testnet-only old-commitment retention |
| `stp.test_broadcast_retained_server_commitment` | Testnet-only old-commitment broadcast |

## Operating Rule

Before every value movement, the Agent reads `stp.safety_snapshot`. It continues only when the channel is `READY_SAFE`, latest commitments exist, commitment height has not rolled back, punishment coverage is `NO_REVOKED_REMOTE_STATE` or `COVERED`, and the target asset is spendable rather than pending.

If the result of a network operation is unknown, the Agent tracks reservation and txids instead of immediately retrying.
