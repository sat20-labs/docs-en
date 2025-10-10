Security of SatoshiNet
====

The security of SatoshiNet is ensured through the following mechanisms:

1. **Lightning channels based on RSMC contracts**: These are native extensions of the BTC mainnet and share the security of the BTC mainnet. This is the most fundamental aspect of security.
  
2. **SatoshiNet based on BTCD source code**: The security at the technical level is provided by BTC technology, ensuring the prevention of double-spending of assets.

3. **Assets and sats all from mainnet**: Every satoshi and asset on SatoshiNet originates from the BTC mainnet, providing asset-level security and preventing the creation or destruction of assets.

4. **Proof of Stake (POS) mechanism on SatoshiNet**: Economically ensures that there is no incentive for malicious behavior among SatoshiNet nodes.

Ultimately, SatoshiNet combines these factors to ensure a robust and rock-solid security model. This security is based on the technology and consensus of the BTC mainnet. As long as the BTC mainnet's technology and consensus remain intact, every satoshi on SatoshiNet remains under the control of the user, and no one can take them away.


Extreme Example
---
Let's use the following extreme example to illustrate why SatoshiNet is secure.

In this extreme example, SatoshiNet shrinks drastically, becoming a single node, SN, a completely centralized node. Think of it as a CEX, if you will. User A/B/C uses their wallet to open a lightning channel with the SN node. Here, we need a solid foundation of consensus: the lightning channel is secure and the assets held by A/B/C and its partners are safe. At any time, everyone can retrieve their funds by broadcasting a commitment transaction.

Now, let's move on to the issue of A/B/C using the SN node to conduct multi-party transactions. Let's consider SN as a CEX, with A/B/C as users of the CEX, and they trade through this specialized CEX. Note that unlike a real CEX, user funds are held in the lightning channel and controlled by their own signatures. Now, if A/B/C begins trading frantically, after a period of time, A/B/C will inevitably find themselves in the following situations:

1. Compared to the initial state of the channel, A's funds are reduced. Because all funds remain in the channel, A maintains control of his assets by holding commitment transactions.  
2. B makes a fortune, and his book assets far exceed the channel's capacity. This means that much of his assets are held in other people's channels. B cannot reclaim control through committed transactions, and therefore feels insecure.

How does SatoshiNet handle this situation? How does it ensure that B maintains control of his assets? It's actually quite simple: it involves the splicing channel adjustment technology. SN nodes use splicing to dynamically adjust the status of each channel to rebalance it. For example, they might reduce the capacity in A's channel and transfer these assets to B's channel. Both parties in the channel can initiate a splicing operation. Note that splicing only affects their own local assets and does not affect remote assets. This is the fundamental principle of remote splicing. Deposits and withdrawals are both performed through splicing. Now, with the support of splicing technology, SN is more like a CEX.

However, it's important to note that splicing is a complex mainnet operation that consumes a significant amount of Satoshi in fees. To reduce fees, SN only adjusts channels when necessary. It also supports user operations by providing a liquidity pool, further reducing the likelihood of channel adjustments. This is somewhat similar to the traditional Lightning Network's submarine operation. Of course, if the amount being adjusted is large, such as adjusting one BTC, a 100,000 satoshi fee might not be too concerning. However, if the asset being adjusted is only 10,000 or 20,000 satoshis, a 10,000 satoshi fee might be quite painful.

Now, it's clear that even if SatoshiNet degenerates into a standalone version, it will remain a trustless, next-generation CEX: it doesn't hold user assets, yet it supports high-frequency, low-friction transactions. This is significantly more advanced than many existing CEXes.

Furthermore, SatoshiNet isn't a standalone chain. Built on the powerful Bitcoin source code, it has the potential to become a more robust BTC layer-2 network than other POS public chains. The standalone version of SatoshiNet already ensures the security of user assets. As SatoshiNet nodes grow and its governance mechanisms become more sophisticated, the network will become increasingly secure. We believe that sooner or later, trust in SatoshiNet will be sufficient to allow the majority of small transactions to be conducted exclusively on SatoshiNet, with only a small number of large-value flows requiring channel capacity updates. These operations will likely occur only within the most important super channels. By that time, the channel will most likely no longer be a two-party channel, but at least a three-party channel, one of which is the SatoshiNet Foundation, which ensures the correct operation of the channel protocol.