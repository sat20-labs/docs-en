# Get Test Assets and Test GAS

Testnet contract interactions require test GAS. SatoshiNet smart contracts are open for testing on the public testnet. Claim test GAS in SAT20 PWA Wallet, then go to `Tools → Smart Contracts` to deploy or invoke template contracts, Prediction contracts, or EVM samples.

Test assets and test GAS are only for public testnet validation. They have no mainnet value and should not be traded as real assets.

## Claim Test GAS

1. Install or open [SAT20 PWA Wallet](https://sat20.org/pwa/).
2. Switch to SatoshiNet testnet.
3. Open `Tools → Faucet`.
4. Select the faucet that supplies test GAS.
5. Claim test GAS and wait for the faucet transaction to confirm.
6. Check whether the wallet shows test GAS balance.

If the faucet page shows an error, keep the wallet address, network, error message, and screenshot, then retry later or report it.

## Where Test GAS Is Used

Test GAS is used for:

1. Deploying a contract.
2. Invoking a contract.
3. Joining Prediction betting.
4. Executing template contracts or EVM samples.
5. Paying Result TX and execution-related fees.

If a contract operation fails, first check:

1. Whether the wallet is on testnet.
2. Whether test GAS is sufficient.
3. Whether the contract address is correct.
4. Whether the transaction generated a txid.
5. Whether the Explorer or Indexer can find the transaction.

## Test Assets

Different tests may require different assets:

1. Prediction contracts can use testnet-supported bet assets.
2. AMM, limit order, Launchpad, and other template contracts may require testnet assets, test GAS, and corresponding contract parameters.
3. EVM sample contracts may require funding assets transferred to the contract address in the Call TX.
4. STP tests may require BTC testnet4 sats and protocol assets on L1.

See the [Contract Template Catalog](../build/contract-template-catalog.md) and [Prediction Contract Test](prediction-contract.md) for testing options.

## Verification

After claiming or using test assets, check:

1. Wallet balance.
2. Txid.
3. SatoshiNet Explorer.
4. L2 Indexer asset state.
5. Contract page or contract state view, if the operation involves a contract.

If wallet display and Explorer / Indexer evidence disagree, treat Explorer / Indexer evidence and txid as the priority for debugging, and avoid repeating value-moving operations until the state is clear.

**Page Status: Implemented / Faucet and test-contract configuration changes with each testnet reset**
