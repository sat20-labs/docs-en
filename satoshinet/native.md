The Native Nature of SatoshiNet
====

The BTC mainnet does not support Turing-complete smart contracts, due to concerns over the risks that activating OP_CAT could introduce, which are perceived to far outweigh any potential benefits. This makes it virtually impossible to activate OP_CAT on the BTC mainnet. As a result, the development trajectory of the BTC ecosystem is fundamentally different from that of the ETH ecosystem.

ETH has Turing-complete smart contracts, which allows the Layer 1 network to verify transaction data from Layer 2 networks. This makes it possible for the assets and transactions on ETH’s Layer 2 networks to have their security ensured by the Layer 1 mainnet. However, this possibility does not exist in the BTC ecosystem. Simply put, the BTC mainnet is a "walled garden" that cannot accept external transaction or asset data. In other words, BTC’s assets can only exist and be settled on the mainnet. Any asset or network that operates outside the mainnet for settlement is not considered part of BTC's native Layer 2 ecosystem. By this standard, the only truly native Layer 2 network for BTC is the Lightning Network.

However, the traditional Lightning Network does not support the native assets currently found on the BTC mainnet, such as Ordinals NFTs, BRC20 tokens, Runes, and others. This renders the Lightning Network as a native second-layer extension somewhat misnamed.

SatoshiNet represents an evolution of the traditional Lightning Network in a new direction. The traditional Lightning Network operates with a combination of RSMC + HTLC protocols, while SatoshiNet abandons HTLC in favor of RSMC + a parallel BTC network, creating a new generation of the Lightning Network. Compared to the traditional Lightning Network, SatoshiNet offers several advantages:

1. **Dynamic Lightning Channels**: Traditional Lightning channels are represented by a single UTXO, whereas the SatoshiNet Lightning channels use a set of UTXOs. These can be dynamically adjusted with splicing techniques to increase or decrease UTXOs or adjust the capacity of individual UTXOs.
  
2. **Supports BTC Native Assets**: While the traditional Lightning Network does not support mainstream asset issuance protocols on the BTC mainnet, such as Ordinals, ORDX, BRC20, and Runes, SatoshiNet supports all of these protocols.

3. **Lightweight Design**: Current popular Lightning Network versions, like LND, lack a lightweight version and cannot be compiled into WASM modules to run in browsers. However, SatoshiNet’s Lightning channel modules can be compiled into WASM, allowing them to run in web browsers.

Beyond these differences, SatoshiNet’s Lightning Network module fully inherits the RSMC protocol, which is the backbone of Lightning Network channel security. This protocol ensures that users can always broadcast commitment transactions to reclaim their assets, and it enables the construction of penalty transactions to deal with malicious counterparty behavior. These mechanisms are also implemented in SatoshiNet, ensuring the security of users' assets.

Moreover, SatoshiNet's features stem from its natural extension of the BTC mainnet:

1. **SatoshiNet’s assets come from the mainnet**.
2. **SatoshiNet’s addresses match those of the mainnet**, providing users with a seamless experience.
3. **SatoshiNet’s network fees are paid in BTC**.
4. **SatoshiNet uses the same UTXO model**, as its code is derived from the BTC source code.

In conclusion, by inheriting the native characteristics of Lightning channels and maintaining consistency with the parallel BTC network, SatoshiNet is a truly native BTC extension network.