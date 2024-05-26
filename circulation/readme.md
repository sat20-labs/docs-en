SAT20 Asset Circulation Protocol
====

The SAT20 Asset Circulation Protocol establishes specifications for basic operations such as entering, exiting, and circulating satoshis within the layer-2 network.

What is the BTC native layer-2 network? We believe there is only one standard: users must have full control over the security of their funds throughout the entire process, without requiring assistance or permission from any party, and have the unilateral ability to decide whether to enter the layer-2 or return to layer-1. Only such a network can be considered a BTC native layer-2 network. The SAT20 Asset Circulation Protocol itself is not a layer-2 network; it is a protocol that governs how satoshis enter the layer-2 network, how they are traded and transferred within the layer-2 network, and how they can be securely returned to layer-1. The SAT20 Asset Circulation Protocol is designed to adapt to all BTC native layer-2 networks, whether existing or future ones.

The core concepts of the SAT20 Asset Circulation Protocol include the following:

1. Satoshi locking and unlocking: Satoshis are locked on the mainnet through channel technology. Locked satoshis are automatically unlocked upon meeting certain conditions or can be manually unlocked by the user, ensuring the security of user funds.
2. Satoshi mapping: Locked satoshis are mapped to the layer-2 network, where they continue to circulate.
3. Satoshi swapping: Satoshi swapping is the core technology that enables seamless circulation within the layer-2 network. Details of this technology are not disclosed at this time.
4. Enhanced UTXO model: In the layer-2 network, the enhanced UTXO model allows additional information about satoshis to be stored within enhanced UTXOs.
5. Client-side verification: The authenticity of any satoshi asset can be verified using its ordinal number and asset issuance records.
6. SVM (Satoshi Virtual Machine): The Satoshi Virtual Machine maximizes the capabilities of satoshis using WebAssembly (WASM) technology.

Locking, unlocking, and mapping to the layer-2 network require a mature channel technology. Essentially, these channel technologies are constructed using BTC script languages. SAT20 can utilize any proven and secure scripts to accomplish the locking and unlocking tasks for satoshis. So far, the most mature channel technology is the Lightning Network's channel technology. Therefore, we have implemented the SAT20 asset circulation protocol on top of the Lightning Network channel technology and demonstrated the protocol's capabilities through a sample project.