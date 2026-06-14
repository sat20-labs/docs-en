# Use: Using SatoshiNet

The Use section is for ordinary users, asset holders, and community members. Its goal is to explain how to enter SatoshiNet, use assets, exit, and judge whether assets remain under user control.

## Recommended Path

1. Install [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. Create or import a wallet.
3. Connect to the default Core Node.
4. Open an STP channel.
5. Use the indexer to confirm Bitcoin L1 asset UTXOs, confirmations, and protocol state.
6. Splice Bitcoin L1 assets into the channel.
7. Use unlock to release assets to a personal SatoshiNet address.
8. Transfer, trade, or use contracts on SatoshiNet.
9. When returning to Bitcoin L1, use lock, splicing-out, or close.

## What Users Need to Understand

1. SatoshiNet assets are not custodial bridge assets.
2. The indexer is the asset fact layer that helps verify whether assets are on Bitcoin L1, in a channel, or on SatoshiNet.
3. Users need to verify that they hold the latest commitment transaction and exit path.
4. Any mainnet value movement should go through wallet authorization.

## Common Scenarios

| Scenario | Read |
| --- | --- |
| I want to move assets into SatoshiNet | [STP Introduction](../learn/stp.md) |
| I want to understand how assets are identified | [Indexer: Bitcoin Asset Fact Layer](../learn/indexer.md) |
| I want to confirm asset safety | [Asset Safety Model](../learn/security-model.md) |
| I want an Agent to help me operate | [AI Agents and User Asset Control](../learn/ai-agent.md) |
| I want to see the testnet drill | [SAT20 Agent Wallet Testnet Validation Record](../ai/sat20-agent-wallet/testnet-validation.md) |

## Risk Boundary

Before mainnet operations, users need to confirm:

1. Whether the network is mainnet or testnet.
2. Asset name and amount.
3. Target address.
4. Fees.
5. Whether L1/L2 indexers can query the relevant transactions and asset state.
6. Whether channel state is safe.
7. Whether the wallet shows an authorization prompt.

If the wallet or Agent cannot prove channel safety, stop the operation.
