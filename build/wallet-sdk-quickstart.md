# Integrate Wallet SDK

Wallet SDK lives in [`sat20wallet/sdk`](https://github.com/sat20-labs/sat20wallet/tree/main/sdk), with module `github.com/sat20-labs/sat20wallet/sdk`. Both the PWA and Transcend use it, but APIs evolve quickly: pin a commit and record local `replace` dependencies instead of relying on a floating version.

## Choose an Integration Layer

| Scenario | Recommended entry point |
| --- | --- |
| Users, DApps, and AI agents | SAT20 PWA Wallet and its authorization / DApp Connect boundary |
| Go wallets or services | Public `sdk/wallet.Manager` interfaces |
| Browser wallets | `sdk/wasm` wrappers; do not bypass authorization from JavaScript |
| STP services | Transcend or an equivalent protocol client reusing SDK channel capabilities |

## Initialization Order

1. Explicitly configure `common.Config`: Env, Chain, Mode, peers, and L1 / L2 Indexers.
2. Isolate persistent data per network. Native Go and WASM use different storage entry points; record the actual backend or browser storage and do not mix Indexer databases with wallet replicas.
3. Check `wallet.NewManager(cfg, db)` before calling `Start()` to start Indexer clients, status bootstrap, action monitoring, channel heartbeat, watchtower, and DKVS sync.
4. Create or import a wallet, then unlock it. Keep the Manager alive while the UI is locked; call `Stop()` / `Close()` when exiting the runtime.
5. Do not mutate chain / environment in place: stop the old Manager and create a new one with the target configuration and an independent DB.

## Business Boundaries

- Asset queries: distinguish Bitcoin, SatoshiNet, and channel positions through L1 / L2 Indexers. RGB11 balances merge client-validated state.
- Sending and broadcasting: select the network explicitly. Repeated broadcast of the same raw transaction can be idempotent success; lower layers must not guess Bitcoin versus SatoshiNet.
- Channels: use `GetCurrentChannel` for the selected wallet / subaccount's single channel. Do not concurrently restore, reopen, or synchronize an old snapshot during a pending lifecycle operation.
- DKVS: domains use SDK DKVS manager / account-management interfaces, without maintaining their own cursor, outbox, or second typed recovery helper.
- Current DKVS uses Active sync/watch and endpoint-local generation, without mutation logs or delete floors. Request signatures bind CAS and current generation; unknown network outcomes permit only original-request replay, without automatic reauthorization. See [DKVS](../protocol/dkvs/readme.md) for the protocol.
- Account recovery: Account Management restores root-account data. Importing a non-root mnemonic only restores that wallet's on-chain data.
- Authorization: private keys, mnemonics, and signing stay within the wallet. DApps and agents submit intent and receive authorized results and evidence.

## Minimum Acceptance

1. Create / import, wrong passwords, password changes, restart / unlock, and duplicate-wallet restrictions.
2. L1 / L2 asset queries, actual sends, and idempotent handling of unknown outcomes.
3. Open / close, splicing, unlock / lock, lock-with-expand for insufficient capacity, and ping recovery.
4. DKVS create, update, delete, and recreation; FREE_LOCAL does not relay, while AUTOPAY updates are eventually readable at another node.
5. Account recovery both without and with RGB11 data.
6. PWA rejection never broadcasts; approval broadcasts once and leaves an operation log.

Local integration test setup and execution instructions are in `sat20wallet/pwa/scripts/verify/README.md`.

See the [API source map](api-source-map.md) and [third-party STP client guide](../protocol/stp/client-integration.md).

**Page Status: Implemented / API Iterating**
