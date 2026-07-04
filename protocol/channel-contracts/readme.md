# Channel Contracts

Channel Contracts are public protocol facilities used by early SatoshiNet to coordinate actions between Bitcoin L1 and SatoshiNet L2. They are different from private STP channels and different from SatoshiNet smart contracts.

A private STP channel manages asset control between two peers, and the assets in that channel can be allocated by the latest commitment state. A Channel Contract manages a public asset pool or public business state. Assets in a Channel Contract do not belong to either peer; they should be understood as belonging to a SatoshiNet public facility. Some assets may be user deposits that users can reclaim according to contract rules.

## Core Positioning

The main purpose of a Channel Contract is to coordinate user-triggered L1/L2 actions:

1. A user sends Bitcoin L1 assets to the contract channel address.
2. The contract identifies the user-triggered action.
3. The contract executes distribution, launch, refund, withdrawal, or state updates on SatoshiNet.
4. When moving from L2 back to L1, the contract coordinates descend / de-anchor and L1 outputs.
5. The contract records each user action, result transaction, and queryable state.

A Channel Contract does not provide private-channel commitment safety for public pool assets. Public pool assets are not split into "local peer" and "remote peer" ownership, and users do not hold a latest commitment that can unilaterally recover the whole pool. It is a protocol-constrained public cross-layer facility.

## Private STP Channels vs Channel Contracts

| Dimension | Private STP Channel | Channel Contract |
| --- | --- | --- |
| Asset ownership | Assets belong to the two channel peers and are allocated by the latest commitment state | Assets belong to a public facility or pool; user deposits are reclaimed by contract rules |
| Safety mechanism | RSMC, commitment transactions, revocation, CSV, punishment, force close | Protocol rules, contract state, co-signing, L1/L2 transactions, and indexer evidence |
| User exit | User holds latest commitment and can force close | User withdraws, refunds, closes, or waits for processing by contract rules |
| Main actions | open, splicing, unlock, lock, close, punish | deploy, invoke, deposit, withdraw, launch, refund, close |
| Goal | Protect asset control between a user and a Core Node | Coordinate public asset transcendence, launches, and public pool state |

Therefore, Channel Contracts should not be described as a business layer where user assets are still protected by private-channel commitments. That safety meaning belongs to private STP channels, not Channel Contract asset semantics.

## Channel Contracts vs Smart Contracts

| Dimension | Channel Contract | Smart Contract |
| --- | --- | --- |
| Main position | Near the L1/L2 connection, around channel addresses and cross-layer actions | Global SatoshiNet execution environment |
| Main purpose | Coordinate L1/L2 asset actions and process user requests in public facilities | Run programmable application state machines on L2 |
| State source | Contract runtime state, invoke history, L1/L2 transactions, ascend/descend records | VM state, canonical Result TX, block state root |
| Asset control | Public pool assets and user deposits processed by contract rules | Contract address assets spent by VM-authorized results |
| Suitable scenarios | Asset transcendence, launches, public cross-layer facilities | AMM, limit order, prediction, EVM, natural language contracts |

The core evolution is the verification scope: Channel Contracts are mainly advanced by the two channel peers around channel state, co-signing, and L1/L2 actions; SatoshiNet smart contracts enter whole-network consensus, where the whole SatoshiNet validates contract invocation, canonical Result TX, and contract state root.

The current AMM and limit order features in PWA Market are still L2 market Channel Contract capabilities, not SatoshiNet smart contracts. The testnet also has smart contract template AMM / LimitOrder and EVM `ConstantProductAMM` / `LimitOrderBook` samples. They can only be accessed through PWA `Tools -> Smart Contracts` and are used to validate the whole-network consensus contract path.

As SatoshiNet smart contracts mature, AMM, limit order, swap, and similar application logic should move toward smart contract templates. Channel Contracts should converge on core L1/L2 coordination facilities such as `transcend.tc` and `launchpool.tc`.

## Core Channel Contracts

### `transcend.tc`

`transcend.tc` is an asset transcendence Channel Contract. It targets L1/L2 entry and exit for arbitrary assets, especially public transcendence scenarios.

Its core semantics:

1. Support specified assets moving between Bitcoin L1 and SatoshiNet L2.
2. When a user deposits, the L1 asset is recognized and processed by the Channel Contract, and the user receives the corresponding L2 distribution.
3. When a user withdraws, the contract coordinates L2 burn or lock and produces the L1 asset output.
4. The contract maintains public-pool L1 UTXOs not yet ascended, L2 available assets, and required descend / de-anchor actions.
5. If the L2 public pool has enough assets, it can distribute directly; if not, it coordinates ascend.
6. If L1 UTXOs are insufficient for withdrawal, it coordinates descend / de-anchor first.

The focus of `transcend.tc` is public cross-layer asset capability, not order matching or private channel safety.

### `launchpool.tc`

`launchpool.tc` is an asset launch Channel Contract. It organizes asset deployment, minting, L2 anchor, user mint participation, distribution after launch thresholds, and refunds or close paths.

Its core semantics:

1. Deploy asset launch parameters such as protocol, ticker, supply, per-address limit, launch ratio, reserved ratio, and binding-sat parameters.
2. Execute L1 deploy and mint actions for protocols such as ORDX, BRC20, and Runes.
3. Bring minted assets into SatoshiNet so the launch pool can manage L2 assets.
4. Record user mint / participation actions on SatoshiNet, including valid and invalid inputs.
5. Execute launch when threshold or expiry conditions are met and distribute assets by rule.
6. Refund invalid inputs or failed scenarios.
7. Return remaining assets to the deployer or participants if the launch fails or the contract closes.
