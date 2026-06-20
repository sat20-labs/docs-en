# Use an AI Agent

AI Agents can help users understand asset state, check STP channel safety, explain transaction purposes, and call wallet adapters after user authorization.

Agents are not wallets. Agents do not store private keys, seed phrases, or wallet passwords.

## Recommended Use

1. Install and initialize SAT20 PWA Wallet first.
2. Create, import, back up, and unlock wallet inside the wallet.
3. Install SAT20 Agent skill.
4. Let the Agent call `wallet.status`.
5. If STP is involved, call `stp.status` and `stp.safety_snapshot`.
6. The Agent provides a safety conclusion and next-step suggestion.
7. Every value-moving operation is confirmed inside the PWA wallet.

**Page Status: In Development**
