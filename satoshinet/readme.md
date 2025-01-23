SatoshiNet
====

SatoshiNet is the native second-layer extension network of BTC, composed of Lightning Network channels (RSMC protocol) and a parallel BTC network.  
SatoshiNet is native and based on the following core technologies:

1. **Assets are all from Layer 1**: SatoshiNet does not issue its own assets. All assets on SatoshiNet are derived from the BTC mainnet.
2. **Assets are secured by the BTC mainnet**: All assets on SatoshiNet are actually stored in Lightning channels, and the security of these channels is guaranteed by the BTC mainnet.
3. **Users retain control**: At any time, even if SatoshiNet shuts down, users can retrieve their assets without the need for any third-party involvement.
4. **Same consensus**: Same addresses, same assets, same network fees.

SatoshiNet represents a **new direction of evolution for the Lightning Network**. Unlike the original Lightning Network, which evolves into a network architecture using a combination of RSMC + HTLC protocols, SatoshiNet retains the RSMC protocol while abandoning HTLC. This allows the Lightning channel’s full potential to be unlocked, transforming it into a bridge between Layer 1 and Layer 2 networks. At the same time, a parallel BTC network is used as the second-layer network to record the transaction states of Lightning channels. This ensures that changes in Lightning channel states are traceable, auditable, and immutable as UTXO ledger records. Additionally, by activating OP_CAT and potentially more opcodes, SatoshiNet can evolve into a network supporting Turing-complete smart contracts. In this way, SatoshiNet allows the BTC ecosystem to compete with the ETH ecosystem.

SatoshiNet is developed on top of the BTC core code, and compared to the BTC mainnet, it mainly differs in the following aspects:

1. **Consensus Mechanism**: Proof of Stake (POS)
2. **Block Time**: 12 seconds
3. **Transaction Network Fees**: 10 satoshis
4. **Enhanced UTXO Model**: Supports explicit expression of any asset
5. **Supports Turing-complete Smart Contracts**: Activates OP_CAT and other instructions, turning Bitcoin scripts into Turing-complete scripts

SatoshiNet is also constantly evolving. All technologies that cannot be implemented on the mainnet can be realized on SatoshiNet. One day, it will even be able to connect to all other public chains through SatoshiNet, making the BTC mainnet truly the foundation of the value Internet.

SatoshiNet is also the ultimate goal of the SAT20 protocol, providing a secure, fast, and cost-effective native circulation environment for BTC and the native assets on the mainnet. SatoshiNet opens up entirely new possibilities for the development of the BTC ecosystem.