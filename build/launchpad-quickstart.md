# Deploy Launchpad

LaunchPool/Launchpad channel-contract runtime and tests exist, but the current PWA release does not expose a standalone public Launchpad deployment entry. Asset deployment or completed minting may trigger existing market-side flows; that alone does not establish a public developer-facing Launchpad product.

Before release, fix the asset and mint prerequisites, peers, funding asset, price and limits, success/failure/refund behavior, AMM handoff, administrator permissions, testnet registry, and asset reconciliation. Missing contract assets must be diagnosed through peer synchronization and deployment state rather than fabricated or bypassed.

**Page Status: Runtime Exists / Public Flow Unavailable**
