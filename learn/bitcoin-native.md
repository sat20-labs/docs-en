# Why Bitcoin Needs a Native Extension Network

Bitcoin L1 is secure, stable, and globally verifiable, but it was not designed for high-frequency trading, complex contracts, or low-cost application interactions. As Ordinals, Runes, BRC20, ORDX, and other asset protocols appear, the Bitcoin ecosystem needs a new liquidity environment: one that inherits Bitcoin L1 asset safety while supporting faster, cheaper, and more programmable applications.

That is SatoshiNet's position.

## What "Bitcoin-Native" Means

"Bitcoin-native" is not just a slogan. It has at least four requirements:

1. Assets originate from Bitcoin L1.
2. Asset state can be verified by Bitcoin L1 transactions, UTXOs, and indexers.
3. Users still have an exit path back to Bitcoin L1 in abnormal situations.
4. The network design respects Bitcoin's UTXO, signature, timelock, and verifiable transaction model.

If a network only records balances off-chain, or custodies assets in a centralized bridge and issues mapped assets, it is not the model SatoshiNet is pursuing.

## SatoshiNet's Path

SatoshiNet uses STP to bring Bitcoin L1 assets into channel addresses and create a transferable state on SatoshiNet. After entering SatoshiNet, users can transfer, trade, and use smart contracts in a faster environment. When users want to exit, they can return to Bitcoin L1 through splicing-out, close, or force close.

Safety comes from protocol structure, not from Core Node promises.

## Comparison

| Path | Strength | Main issue |
| --- | --- | --- |
| Direct Bitcoin L1 transactions | Secure and final | Slow, expensive, not suited for complex applications |
| Centralized custodial bridge | Simple UX | Users lose asset control |
| Sidechain or independent chain | Strong programmability | Asset safety depends on external consensus or bridge assumptions |
| Lightning Network | Mature payment use case | Limited multi-asset, contract, and application extension |
| SatoshiNet | Bitcoin-native assets, STP safety exits, contracts and Agents | Requires a stronger wallet, indexer, and developer ecosystem |

SatoshiNet does not replace Bitcoin L1. Its goal is to give Bitcoin L1 assets new liquidity and application space.
