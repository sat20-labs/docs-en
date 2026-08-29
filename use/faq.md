# FAQ

## Does a zero balance mean my assets are lost?

Not necessarily. Verify network, wallet, and subaccount. Zero is valid before any local cache exists; trusted cache should remain visible if a remote refresh fails.

## Can I retry after a transaction timeout?

Not immediately. Timeout, EOF, reload, or restart may happen after broadcast. Check logs, reservations, txid, mempool, and Indexer first. Rebroadcasting identical raw bytes may be idempotent; constructing another business transaction is different.

## Why is funding still pending after two minutes?

Funding monitoring continues after broadcast until confirmation. Short timeouts apply only to pre-broadcast negotiation/RPC.

## Why can the wallet not restore or reopen while a reservation is pending?

Pending means the lifecycle is unresolved. The wallet must not restore another runtime from ping or reopen before the old channel is fully closed.

## Is the Market AMM the same as the Tools AMM?

No. Market AMM/limit orders are L2 Market Channel Contracts. AMM/LimitOrder under `Tools -> Smart Contracts` are SatoshiNet smart-contract templates.

## Is DKVS a wallet task cache?

No. DKVS stores important small data such as account recovery, RGB11 encrypted snapshots, mailbox messages, and app configuration.

## Does mainnet support rollback or fault injection?

No. Mainnet nodes must reject testnet rollback and revoked-state fault-injection interfaces.

**Page Status: Maintained**
