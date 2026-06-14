# Asset Issuance Protocols

Bitcoin L1 natively understands only BTC and UTXOs. Ordinals, Runes, BRC20, ORDX, and other protocols write asset meaning into Bitcoin transactions, scripts, inscriptions, sat ranges, or protocol events. The indexer does not invent those assets; it parses them from chain facts into unified, queryable, auditable asset state.

Asset issuance protocols should therefore be understood under the Indexer section. They are the protocol families the indexer must support and express uniformly. ORDX is a SAT20-designed sat-bound asset issuance protocol, but it is not the only asset protocol. SatoshiNet needs to serve assets that already exist on Bitcoin L1 and those that will appear later.

## Main Asset Types Supported by Indexers

| Protocol / asset type | Core meaning | What the indexer expresses |
| --- | --- | --- |
| BTC | Native Bitcoin UTXO asset | Address UTXOs, amount, confirmations, spent state, mempool / reorg state |
| Ordinals | Digital objects based on sat numbers and inscriptions | sat range, inscription, owner, containing UTXO, transfer history |
| Runes | Fungible asset protocol on Bitcoin L1 | etching, mint, transfer, balances, UTXO ownership, validity |
| BRC20 | Inscription-based deploy / mint / transfer asset protocol | ticker, deploy, mint, transfer inscription, valid/invalid transfer, address balance |
| ORDX | SAT20 sat-bound asset issuance protocol | ticker, deploy, mint, bound sats, unbind, freeze, UTXO ownership, sat range |

STP and SatoshiNet confirm asset facts through indexers instead of guessing assets directly. Before an asset enters a channel, the client must use the L1 indexer to confirm protocol, amount, validity, and confirmation state. After assets enter SatoshiNet, the L2 indexer tracks ascend, descend, enUTXO, channels, and contracts.

## Unified Expression Principles

Indexers should provide unified query semantics across asset protocols:

1. Asset identity: protocol, ticker / rune id / inscription id / asset name.
2. Asset amount: expressed with protocol precision without losing divisibility.
3. Carrier location: txid, vout, sat range, address, layer.
4. Validity: whether the protocol event is valid and whether it has been consumed or invalidated by later events.
5. Confirmation state: mempool, confirmations, height, reorg risk.
6. Cross-layer state: whether the asset has ascended to SatoshiNet or descended back to Bitcoin L1.

Wallets, exchanges, STP clients, and AI Agents should base decisions on these fields rather than a single balance field.

## ORDX Protocol

ORDX is SAT20's sat-bound asset issuance protocol. Its core idea is that assets are not independent account balances; they are bound to identifiable and traceable sats. Where the sat goes, the asset goes. Whoever owns the sat owns the asset.

### Core Properties

ORDX inherits several key properties from sats:

1. Sats cannot be destroyed arbitrarily, so assets bound to sats cannot disappear outside protocol rules.
2. Sat numbering makes each sat identifiable, so assets can bind to specific sats or sat sets.
3. Sats move in UTXOs, and assets move with those UTXOs.
4. Sat non-fungibility gives assets a natural SFT property.
5. Issuance, minting, unbind, freeze, and other states are recomputed by indexers from Bitcoin L1 transactions.

ORDX is different from account-model assets. Wallets and exchanges cannot only read address balances; they must understand carrying UTXOs, sat ranges, binding amounts, and protocol state.

### Basic Capabilities

ORDX depends on:

1. Identifying and tracking sats: which sat ranges a UTXO contains and where a sat currently is.
2. Writing and reading protocol data on sats: using inscriptions or OP_RETURN to express deploy, mint, unbind, freeze, and other events.

The indexer is the ORDX fact computation layer. It parses Bitcoin L1 transactions, determines event validity, and provides query results to wallets, explorers, STP clients, and exchanges.

### Deploy

`deploy` deploys an ORDX ticker.

Key fields include `p=ordx`, `op=deploy`, `tick`, `lim`, `n`, `selfmint`, `max`, `block`, `attr`, and `des`.

Example:

```json
{
  "p": "ordx",
  "op": "deploy",
  "tick": "satoshi",
  "block": "830000-833144",
  "lim": "10000"
}
```

Deploy validation includes ticker uniqueness, block-window checks, field formats, supply limits, name length, and sat attribute conditions.

### Mint

`mint` mints an already deployed ticker.

Example:

```json
{
  "p": "ordx",
  "op": "mint",
  "tick": "satoshi"
}
```

Mint validation includes deployed ticker existence, per-mint limit, total supply, block window, self-mint rules, and sat attribute requirements.

## ORDX v2 Instructions

ORDX v2 extends the legacy version with capabilities better suited to L2 circulation and controlled-asset scenarios.

### OP_RETURN Data

v2 supports writing protocol data through OP_RETURN:

```text
OP_RETURN | SAT20_MAGIC_NUMBER | CONTENT_TYPE | CONTENT
```

This makes some protocol events more compact and easier for indexers to parse.

### Unbind

`unbind` permanently separates a specified ORDX ticker from sats in a target UTXO output. It can only be initiated by the asset owner.

### Freeze / Unfreeze

`freeze` and `unfreeze` support address-level control for scenarios such as stablecoins or controlled assets. Frozen assets cannot be used as valid transferable assets for that ticker. If the underlying Bitcoin UTXO is spent while frozen, the frozen asset does not transfer with outputs; it becomes invalid at the protocol layer.

## Relationship With STP and SatoshiNet

Before assets enter SatoshiNet, the L1 indexer must confirm protocol type, amount, validity, and UTXO location. STP splicing-in only ascends one asset explicitly specified by the user. If a UTXO carries multiple assets, unspecified assets do not ascend.

ORDX must bind sats. When entering SatoshiNet, the client must calculate required sats using `bindingSat`. ORDX mint outputs may also contain Ordinals NFTs; current STP does not ascend `ordx:o` objects to SatoshiNet, and clients/indexers treat them as attached objects that do not enter SatoshiNet.

BRC20 and Runes do not need bound sats on SatoshiNet, but they still obey their own Bitcoin L1 protocol and output rules on L1. STP clients and Agents cannot apply SatoshiNet 0-sat UTXO rules to L1.

## Related Services

ORDX can extend into name services, KV data services, SFT/NFT/FT/DID, and other application forms. They all share the same foundation: asset facts are recomputed from Bitcoin L1 transactions and indexers, and asset control is determined by UTXO ownership.
