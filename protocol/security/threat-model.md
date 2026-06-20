# Threat Model and Trust Assumptions

SatoshiNet safety is not a balance number. It is a set of verifiable exit evidence and trust boundaries.

## Core Trust Assumptions

1. Bitcoin L1 provides final settlement and dispute boundaries.
2. Indexers need cross-verification; a single indexer response is not final safety proof.
3. STP safety depends on valid channel state, commitment transactions, revocation material, punishment coverage, and wallet backup.
4. SatoshiNet nodes handle execution and block production, but do not replace Bitcoin L1 as the final boundary.
5. Wallets control private keys, seed phrases, authorization, and critical local state.
6. Agents can only read evidence and call adapters; they cannot store private keys or bypass signatures.

## Main Risks

1. Core Node offline or refusing service.
2. Core Node broadcasting an old commitment transaction.
3. Wallet losing local state or backup.
4. Indexer divergence, unindexed state, or reorg.
5. Bitcoin L1 reorg.
6. SatoshiNet stopping block production.
7. Contract vulnerabilities.
8. Confusion between public channel contracts and private STP channel safety models.

## What Is Not Guaranteed

1. SatoshiNet state is not unconditionally guaranteed by Bitcoin.
2. Indexers do not take responsibility for user private keys or authorization.
3. Agents do not make final signing decisions for users.
4. Contract assets and private channel assets have different safety boundaries.
5. Testnet validation does not mean mainnet has no risk.

**Page Status: Planning**
