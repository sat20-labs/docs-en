# I Am an AI Agent Developer

AI Agents have two product lines in the SatoshiNet ecosystem.

The first is Agent Wallet & Safety: helping users safely operate wallets, STP channels, and cross-layer asset movement.

The second is Community Builder Agent: helping BTC communities design and gradually deploy their own DEX, DAO, Indexer, Explorer, wallet entry, and contract modules through conversation.

## Agent Wallet & Safety

1. Agents do not store private keys, seed phrases, or wallet passwords.
2. Agents call wallets through PWA Wallet adapters.
3. Agents read `stp.safety_snapshot` before value-moving operations.
4. Agents check commitments, punish coverage, L1/L2 indexer evidence, and pending transactions.
5. Agents stop operations when network results are unknown, punishment coverage is missing, or channel safety degrades.

## Community Builder Agent

The goal is to let any BTC community design and gradually deploy its own DAO, DEX, wallet, Indexer, and Explorer through a few rounds of conversation.

**Page Status: In Development**
