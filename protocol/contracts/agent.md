# Natural Language Contracts

Natural Language Contracts are SatoshiNet smart contracts assisted by AI Agents. Contract content is expressed as natural-language protocol text and interpreted into structured, verifiable, executable results by the protocol Agent.

Natural Language Contracts follow the common [Smart Contract Protocol](./). This document defines their boundary and first-stage requirements.

## Naming

Recommended naming:

1. Chinese name: 自然语言合约.
2. English name: Natural Language Contract.
3. Technical shorthand: Agent Contract.

Protocol documents use "Natural Language Contract". Code-level names may use "Agent".

## Contract Type

Natural Language Contracts are not EVM contracts, do not require Solidity, ABI, or bytecode, and are not template contracts embedded in node source code. They are also not channel contracts and do not use RSMC channels, peer multi-signature state, or commitment transactions.

## Core Node Agent

The first stage treats the SatoshiNet network as having one protocol Agent executed by the built-in Core Node. Core Node identity is checked through the common caller rule: validators derive the caller from the last input's previous output address and require that address to equal the configured Core Node address for Core Node-only actions such as `ready`, `reject`, and `confirm`.

Core Node confirmation payloads do not repeat Core Node public key or signature fields. The confirmation transaction must be sent by the valid Core Node address, and Result TXs and state roots are still independently verified by all nodes. This keeps Agent identity checks aligned with other contract calls and avoids redundant identity fields in payloads.

The Core Node Agent acts as:

1. Agent executor.
2. Reviewer of contract understandability, verifiability, and executability.
3. Protocol oracle.
4. Authorized producer of Natural Language Contract execution conclusions.

Ordinary users can deploy contracts, submit materials, request execution, or raise disputes, but they cannot produce protocol-recognized Agent conclusions for Core Node-only actions. Later protocol versions can extend this to multi-Agent verification.

## Contract Content

The contract text must be an executable agreement that an Agent can parse into verifiable and executable structure.

It should define:

1. Purpose.
2. Participants and roles.
3. Assets.
4. How assets enter the contract.
5. Trigger conditions.
6. Verification data sources.
7. Acceptance criteria.
8. Execution method.
9. Timeout, failure, and dispute handling.
10. Asset-transfer outcomes.

Structured helper fields can include asset lists, participant addresses, time or block-height limits, allowed data sources, validator or arbitrator rules, and contract version. The natural-language body remains the semantic source of truth. Structured fields assist parsing but must not change the core meaning.

## Deployment

`CONTRACT_DEPLOY` deploys a Natural Language Contract. The payload includes contract type, contract body, version, deployer, deployment nonce, and optional structured helper fields.

Deployment creates a pending contract. It does not make the contract executable by itself. Every Natural Language Contract must provide a fixed Core Node activation action:

```
ready
```

Only the Core Node can call `ready`. During activation, the Core Node checks that the contract is complete, conditions are verifiable, acceptance criteria are clear, execution can map to asset intent, private unverifiable facts are not required, and the text is not contradictory or impossible to execute.

If accepted, canonical `CONTRACT_RESULT` moves the contract to `Ready`. If rejected, the contract moves to `Rejected` or `Invalid` and cannot produce asset-transfer execution results.

## Invocation

`CONTRACT_INVOKE` submits events, proofs, acceptance requests, execution requests, or dispute materials. The call transaction must output to the contract address as gas/funding. Caller identity is derived from the last input's previous-output address.

Invocations are divided into:

1. Core Node built-in actions, including `ready` and other Agent-only actions.
2. Ordinary business actions submitted by participants or authorized addresses.

Default invocation can represent asset deposit or other simple chain-visible behavior only when the contract protocol explicitly defines it.

## Verification and Execution

The Agent must convert natural-language terms into verifiable data requirements. A Natural Language Contract can use BTC L1 facts, SatoshiNet facts, indexer results, transaction IDs, block height, signatures, and structured materials. It must avoid private facts that cannot be verified by consensus or by the contract's accepted data-source rule.

Asset movement is never executed directly by the Agent. The Agent produces a protocol conclusion. Block producers include the canonical Result TX, and validators replay and verify the conclusion under the contract rules.

## First-Stage Scope

The first stage focuses on prediction-style Agent contracts, and this subtype is testnet-only. On mainnet, Agent contracts currently expose no usable subtype; nodes, oracle services, indexers, and markets should not show or execute prediction contracts as mainnet contracts.

The expected user flow is:

1. User deploys natural-language contract text.
2. Core Node Agent reviews and activates or rejects it.
3. Participants submit materials or execution requests.
4. Core Node Agent submits the structured protocol conclusion from the configured Core Node address.
5. SatoshiNet records the canonical Result TX.

The long-term direction is to support richer verification sources, multi-Agent consensus, dispute mechanisms, and more complex contract types while keeping the same principle: asset movement is driven by verifiable contract results, not by an off-chain Agent transferring funds.

## Prediction Contracts

Prediction contracts are the first fixed subtype of Natural Language Contracts:

```text
prediction
```

They represent an event with multiple candidate outcomes. Users bet by sending the configured betting asset to the contract address through a `bet` invocation. After the event, the Core Node Agent checks the configured data source and submits a structured `confirm` result. The runtime then settles winners or refunds users through canonical Result TXs.

### Deployment Payload

A prediction deployment payload contains:

```json
{
  "subtype": "prediction",
  "title": "...",
  "description": "...",
  "time_base": "unix",
  "event_time": 1780310400,
  "bet_deadline": 1780306800,
  "confirm_after": 1780396800,
  "source_url": "https://example.com",
  "bet_asset": "::",
  "min_bet_unit": "100",
  "outcomes": [
    {"id": "a", "text": "..."},
    {"id": "b", "text": "..."}
  ]
}
```

`time_base` can be `unix` or `height`. `bet_asset` uses the SatoshiNet asset-name string format. `min_bet_unit` is a Decimal string interpreted with the betting asset's precision. Outcome ids are lowercase letters and must be unique.

### Bet

After Core Node activation, the common contract state is `Ready` and the prediction runtime state is `Betting`. A `bet` invocation contains:

```json
{
  "outcome_id": "a"
}
```

The bet amount is derived from the Call TX funding output to the contract address, not from the payload. It must use the configured `bet_asset`, be at least `min_bet_unit`, and be an integer multiple of `min_bet_unit`. The same address can bet on multiple outcomes or add to an existing outcome position.

### Confirm

`confirm` is a Core Node-only action. Its payload contains:

```json
{
  "result_type": "outcome",
  "outcome_id": "a",
  "result_url": "https://example.com/match/result/123",
  "result": "Team A 2-1 Team B",
  "observed_at": 1780314000,
  "agent_version": 1,
  "model_version": "model-v1"
}
```

Rules:

1. `result_type` is one of `outcome`, `cancelled`, `invalid`, or `unverifiable`.
2. For `outcome`, `outcome_id` must exist in the deployment payload.
3. For refund result types, `outcome_id` must be empty.
4. `result_url` must remain within the site scope defined by deployment `source_url`; the scheme must match and the host must be the same host or a subdomain, including after redirects.
5. `result` records the observed real-world result or explanation and is limited to 128 bytes.
6. `observed_at` records the time or height at which the Agent observed the result.
7. `agent_version` is an integer Agent implementation version. `model_version` records the model version.

Consensus validation checks the Core Node caller identity and the structured confirmation result. It does not refetch external web pages during block validation. Explorers and audit tools can use `result_url`, `result`, `observed_at`, and Agent logs to review the confirmation process.

### Settlement and Refund

Prediction contracts do not expose a public `settle` or `refund` action. A valid `confirm` automatically triggers settlement or refund.

When there are winners, the runtime-recorded valid betting pool is split as:

1. 6% to deployer.
2. 3% to Agent.
3. 1% to bootstrap.
4. 90% to winners pro rata by winning bet amount.

If there are no winners, or if the result is `cancelled`, `invalid`, or `unverifiable`, no fee is charged and runtime-recorded valid bets are refunded by original bet amount.

Only valid `bet_asset` amounts recorded by the runtime participate in prediction settlement. Invalid calls, non-betting assets, and assets held at the contract address beyond the runtime-managed record are handled by the common framework rules and do not participate in the prize pool.

### Runtime States

Prediction runtime states include:

1. `Betting`: bets are open.
2. `ClosedForBet`: the betting deadline has passed.
3. `PendingResult`: `confirm_after` has been reached and the Agent should confirm.
4. `Confirmed`: a result has been accepted.
5. `Settled`: payouts or refunds have been settled.
6. `Refundable`: the contract is in a refund path.

`Betting` and the other prediction runtime states are subtype-specific runtime states, not replacements for the common Agent contract states such as `Ready`, `Rejected`, or `Completed`.
