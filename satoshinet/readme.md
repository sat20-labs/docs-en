SatoshiNet
====

SatoshiNet is the native second-layer extension network of BTC, composed of Lightning Network channels (RSMC protocol) and a parallel BTC network.  
SatoshiNet is native and based on the following core technologies:

1. **Assets are all from Layer 1**: SatoshiNet does not issue its own assets. All assets on SatoshiNet are derived from the BTC mainnet.
2. **Assets are secured by the BTC mainnet**: All assets on SatoshiNet are actually stored in Lightning channels, and the security of these channels is guaranteed by the BTC mainnet.
3. **Users retain control**: At any time, even if SatoshiNet shuts down, users can retrieve their assets without the need for any third-party involvement.
4. **Same consensus**: Same addresses, same assets, same network fees.

SatoshiNet is a Lightning Network that has evolved in another direction, or even a second-generation Lightning Network. This is not only because it circumvents and solves the capacity, traffic, routing, and single-point channel state storage issues of the traditional Lightning Network, but more importantly, SatoshiNet has expanded the RSMC framework and proposed the concept of channel contracts on this basis, giving SatoshiNet a special on-chain programming capability, greatly expanding the construction capabilities of the BTC ecosystem and giving SatoshiNet the opportunity to become the most promising and competitive native extension network in the BTC ecosystem.

SatoshiNet is developed on top of the BTCD source code, and compared to the BTC mainnet, it mainly differs in the following aspects:

1. **Consensus Mechanism**: Proof of Stake (POS)
2. **Block Time**: 12 seconds
3. **Transaction Network Fees**: 10 satoshis
4. **Enhanced UTXO Model**: Supports explicit expression of any asset
5. **Enhanced script commands**: Activate OP_CAT or other more powerful commands (not yet implemented)
6. **Channel contracts**: Support more flexible and powerful smart contracts through WASM

SatoshiNet is also constantly evolving. All technologies that cannot be implemented on the mainnet can be realized on SatoshiNet. One day, it will even be able to connect to all other public chains through SatoshiNet, making the BTC mainnet truly the foundation of the value Internet.

SatoshiNet is also the ultimate goal of SAT20Labs. We envision SatoshiNet to grow into the following network:
1. A native extension of the BTC ecosystem: Providing a secure, fast, and economical native circulation loop for BTC and native assets on the mainnet.
2. Making the BTC ecosystem a direct competitor to the EVM ecosystem: By enabling all existing financial applications and services on the EVM through channel contracts, the BTC ecosystem will become a direct competitor to the EVM ecosystem.
3. Making SatoshiNet the most critical infrastructure for DAPPs in the crypto ecosystem: Continuously expanding SatoshiNet's basic functionality will enable a richer and more prosperous DAPP ecosystem based on SatoshiNet.

We will continue to optimize and iterate SatoshiNet, promoting the development of the SatoshiNet ecosystem and, in turn, the BTC ecosystem, making the BTC ecosystem the premier ecosystem in the crypto industry.