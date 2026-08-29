# Integrate Wallet SDK

Wallet SDK lives in [`sat20wallet/sdk`](https://github.com/sat20-labs/sat20wallet/tree/main/sdk) and is used by both the PWA and Transcend. Its APIs still evolve quickly, so integrations should pin a commit and record local `replace` dependencies.

Use `sdk/wallet.Manager` for Go services and `sdk/wasm` for browser wallets. Configure the environment, chain, mode, peers, L1 Indexer, and L2 Indexer explicitly; keep a separate database per network. Start the Manager once, keep it alive while the UI is merely locked, and recreate it only when switching chain or environment.

Key boundaries are explicit Bitcoin-versus-SatoshiNet broadcasting, one current channel per selected wallet/subaccount, SDK-owned DKVS/account management, root-account recovery, and wallet-side authorization. Minimum acceptance covers create/import/password flows, L1/L2 sends, channel lifecycle and recovery, lock-with-expand, DKVS CRUD and relay policy, account recovery with and without RGB11, and single-broadcast approval behavior.

See the [API source map](api-source-map.md) and [third-party STP client guide](../protocol/stp/client-integration.md).

**Page Status: Implemented / API Iterating**
