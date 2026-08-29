# Build a White-Label DEX

Reusable components already exist: SatoshiNet nodes, L1/L2 Indexers, SAT20 PWA DApp Connect, mainnet/testnet market frontends, and AMM and limit-order channel contracts. The repositories do not yet publish a versioned one-click white-label DEX package with an installer, configuration schema, and upgrade policy.

Deployments must isolate domains, CSP allowlists, market URLs, API proxies, Indexer data, contract registries, and test wallets. Test sites must not reuse production domains or production contract configuration. Validate wallet origin approval, network parameters, AMM/limit orders, asset accounting, recovery, Explorer links, and upgrade rollback before release.

**Page Status: Components Available / Packaged Toolkit Planned**
