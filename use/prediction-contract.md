# Prediction Contract Test

Prediction is the first Agent contract scenario opened on the SatoshiNet public testnet. Users can view bettable matches or events in SAT20 PWA Wallet, place bets, and deploy new Prediction contracts from the tools page.

This page is for testnet users. The Prediction contract is currently for testnet demonstration and validation only. It does not represent mainnet availability.

## Before Testing

1. Install or open [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. Confirm that the wallet is connected to the SatoshiNet testnet.
3. Claim test GAS from the PWA `Tools` home page.
4. Wait until the wallet shows that test GAS has arrived.

Contract deployment, betting, result confirmation, and other contract interactions all require test GAS. If a transaction fails, first check whether test GAS is sufficient.

## Place a Bet

1. Open SAT20 PWA Wallet.
2. Go to `Market`.
3. Open `Prediction`.
4. View currently bettable matches or events.
5. Select one match and read the title, outcomes, betting deadline, result source, and minimum bet unit.
6. Choose the outcome you want to bet on.
7. Enter the bet amount.
8. Confirm the transaction and fee in the wallet.
9. Wait for testnet confirmation.
10. Return to `Market -> Prediction` and check the bet status.

The bet amount is determined by the asset transferred to the contract address in the transaction. The selected outcome only expresses betting direction; the asset amount is determined by the transaction constructed and signed by the wallet.

## Deploy a Prediction Contract

1. Open SAT20 PWA Wallet.
2. Go to `Tools`.
3. Confirm that test GAS has been claimed.
4. Open the Prediction contract deployment tool.
5. Enter the match or event title.
6. Enter the match description.
7. Define outcomes, such as home win, away win, draw, or other clearly verifiable results.
8. Set event time, betting deadline, and the time after which the result can be confirmed.
9. Enter the result source URL. The URL should point to a publicly verifiable page.
10. Set the bet asset and minimum bet unit.
11. Review the parameters and submit deployment.
12. Confirm the deployment transaction in the wallet.
13. Wait for testnet confirmation.
14. After deployment succeeds, view the new match in `Market -> Prediction`.

The deployer must ensure that the prediction event can be verified. An unclear result source, incomplete outcomes, unreasonable timing, or unsuitable minimum bet unit may prevent the contract from becoming bettable or from being settled later.

## Result Confirmation and Settlement

Prediction results are confirmed by the testnet Core Node Agent. After the configured confirmation time is reached, the Agent checks the data source specified at deployment and submits a structured result.

Possible results include:

1. One outcome wins.
2. The event is cancelled.
3. The result is invalid.
4. The result cannot be verified.

When there is a clear winner, the contract distributes the prize pool by rule. If no winner can be confirmed, the event is cancelled, or the result cannot be verified, the contract enters the refund path. See [Natural Language Contracts](../protocol/contracts/agent.md) for protocol details.

## How to Verify

Users should not rely only on page balances. Check evidence whenever possible:

1. Whether the wallet has the bet transaction txid.
2. Whether the contract address received the corresponding bet asset.
3. Whether the testnet Explorer can find the deploy, bet, and result transactions.
4. Whether the Prediction page shows the same match and bet status.
5. Whether a corresponding Result TX exists after result confirmation.
6. Whether winning or refunded assets returned to the user address.

If page state, wallet records, and Explorer evidence are inconsistent, stop placing more bets, keep txids and screenshots, and report the issue to SAT20 Labs.

## Risk Boundary

1. The current Prediction contract is testnet-only.
2. Test assets and test GAS have no mainnet value.
3. The testnet may restart, roll back, or change rules.
4. Users should never enter real private keys, mnemonics, or exchange account information.
5. Result source URLs should be publicly accessible pages, not private chats, verbal commitments, or non-reviewable data.

**Page Status: In Development**
