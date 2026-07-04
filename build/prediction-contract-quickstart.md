# Prediction Contract Quickstart

Prediction is the first Agent contract scenario opened on the SatoshiNet public testnet. It validates the basic loop of Natural Language / Agent contracts inside the SatoshiNet smart contract framework: deploy, ready, bet, result confirmation, Result TX, payout, or refund.

This page is for testnet deployers and developers. For ordinary user steps, see [Prediction Contract Test](../use/prediction-contract.md).

## Current Status

| Item | Status |
| --- | --- |
| Contract type | Agent Contract / Natural Language Contract |
| Subtype | `prediction` |
| Environment | SatoshiNet public testnet |
| Mainnet status | Not open yet |
| Recommended entry | SAT20 PWA Wallet `Tools` and `Market -> Prediction` |
| Fee asset | Test GAS |

## Basic Flow

1. The deployer creates a Prediction contract in PWA `Tools`.
2. The deploy transaction enters the SatoshiNet testnet.
3. The Core Node Agent reviews the contract content and calls `ready`.
4. The contract becomes bettable.
5. Users choose an outcome in `Market -> Prediction` and place bets.
6. After the betting deadline, the Core Node Agent confirms the result according to the specified data source.
7. The contract generates a canonical Result TX.
8. Users receive payout or refund.

## Deployment Parameters

A Prediction deployment should include:

| Field | Meaning |
| --- | --- |
| `title` | Match or event title |
| `description` | Event description that helps users understand the prediction |
| `time_base` | Time base, usually unix time, or height if supported by the protocol |
| `event_time` | Match or event time |
| `bet_deadline` | Time when betting stops |
| `confirm_after` | Time when the Agent starts result confirmation |
| `source_url` | Result source URL |
| `bet_asset` | Bet asset name |
| `min_bet_unit` | Minimum bet unit |
| `outcomes` | Outcome list |

Outcomes should be clear, mutually exclusive, and verifiable by the result source. Avoid descriptions such as "maybe", "probably", or "it depends" that cannot lead to a clear settlement result.

## Example Payload

```json
{
  "subtype": "prediction",
  "title": "Team A vs Team B",
  "description": "Predict the final match winner.",
  "time_base": "unix",
  "event_time": 1780310400,
  "bet_deadline": 1780306800,
  "confirm_after": 1780396800,
  "source_url": "https://example.com/match/team-a-vs-team-b",
  "bet_asset": "::",
  "min_bet_unit": "10000",
  "outcomes": [
    { "id": "a", "text": "Team A wins" },
    { "id": "b", "text": "Team B wins" },
    { "id": "c", "text": "Draw" }
  ]
}
```

Field details follow the Prediction protocol definition in [Natural Language Contracts](../protocol/contracts/agent.md).

## Deployment Checks

Before deployment, check:

1. The wallet is on testnet.
2. The wallet has enough test GAS.
3. `bet_deadline` is earlier than `event_time`.
4. `confirm_after` is not earlier than the time when the result can be publicly queried.
5. `source_url` is publicly accessible.
6. There are at least two outcomes, and they do not overlap.
7. `min_bet_unit` matches the target test asset precision and user experience.

## Bet Invocation

When a user places a bet, the invocation parameter only expresses the outcome:

```json
{
  "outcome_id": "a"
}
```

The bet amount is determined by the funding output transferred to the contract address in the Call TX. It should not be duplicated in invocation parameters. This avoids inconsistency between UI parameters and on-chain asset inputs.

## Result Confirmation

Result confirmation is initiated by the Core Node Agent. The confirmation parameters include:

```json
{
  "result_type": "outcome",
  "outcome_id": "a",
  "result_url": "https://example.com/match/team-a-vs-team-b/result",
  "result": "Team A 2-1 Team B",
  "observed_at": 1780314000,
  "agent_version": 1,
  "model_version": "model-v1"
}
```

`result_url` should belong to the website scope specified by `source_url` at deployment. The contract runtime processes the structured result. Indexers, explorers, and audit tools can review the confirmation process through `result_url`, `result`, `observed_at`, and Agent logs.

## Verify Result TX

Developers and testers should check:

1. Whether Deploy TX succeeded.
2. Whether Ready Result TX appears.
3. Whether Bet TX is accepted by the contract.
4. Whether Confirm is initiated by the Core Node Agent.
5. Whether settlement Result TX distributes the prize pool or refund by rule.
6. Whether contract state root updates with the block.
7. Whether wallet, market, and Explorer display the same state.

If contract state and asset distribution are inconsistent, first preserve the contract address, txids, block height, page screenshots, and wallet logs.

## Relationship to Other Contract Types

Prediction is not an EVM contract and does not require Solidity, ABI, or bytecode. It is not a Channel Contract either, and does not rely on verification by two channel peers. Prediction belongs to the SatoshiNet smart contract system and is verified by the whole network through contract invocation, Result TX, and state root.

EVM contracts are suited to reusing the Solidity ecosystem. Template contracts are suited to high-frequency, fixed-rule scenarios that need deterministic node runtime. Prediction is suited to testing how Agents turn externally verifiable events into structured contract results.

**Page Status: In Development**
