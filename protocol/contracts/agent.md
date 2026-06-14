# Natural Language Contracts

Natural Language Contracts are SatoshiNet smart contracts assisted by AI Agents. Contract content is expressed as natural-language protocol text and interpreted into structured, verifiable, executable results by the protocol Agent.

Natural Language Contracts follow the common [Smart Contract Protocol](readme.md). This document defines their boundary and first-stage requirements.

## Naming

Recommended naming:

1. Chinese name: 自然语言合约.
2. English name: Natural Language Contract.
3. Technical shorthand: Agent Contract.

Protocol documents use "Natural Language Contract". Code-level names may use "Agent".

## Contract Type

Natural Language Contracts are not EVM contracts, do not require Solidity, ABI, or bytecode, and are not template contracts embedded in node source code. They are also not channel contracts and do not use RSMC channels, peer multi-signature state, or commitment transactions.

## Core Node Agent

The first stage treats the SatoshiNet network as having one protocol Agent executed by the built-in Core Node. Core Node-generated confirmations must include the Core Node public key and signature. Validators verify that the signer is the protocol-recognized Core Node identity and that the signature covers the relevant contract address and confirmation parameters.

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

```text
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

The first stage focuses on prediction-style Agent contracts. The expected user flow is:

1. User deploys natural-language contract text.
2. Core Node Agent reviews and activates or rejects it.
3. Participants submit materials or execution requests.
4. Core Node Agent signs the protocol conclusion.
5. SatoshiNet records the canonical Result TX.

The long-term direction is to support richer verification sources, multi-Agent consensus, dispute mechanisms, and more complex contract types while keeping the same principle: asset movement is driven by verifiable contract results, not by an off-chain Agent transferring funds.
