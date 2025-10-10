Dynamic Channel Technology - Splicing
====

A traditional Lightning Network channel is a single UTXO, while a Lightning channel in the Satoshi Transcending Protocol is a group of UTXOs. Splicing allows for adding or subtracting UTXOs, adjusting the capacity of a specific UTXO, and even managing a single multi-signature address as an entire Lightning channel at any time. This flexibility is essential for transferring assets from the mainnet to the second-layer network.

1. Multiple UTXOs
2. Arbitrary Addition and Subtraction of UTXOs
3. Arbitrary Adjustment of UTXO Size

The Circulation Protocol uses dynamic channel technology to arbitrarily adjust channel capacity, allowing the channel to pass any type of asset at any time.

Splicing is an atomic operation. Each splicing action is completed by multiple actions, forming an atomic operation. If one action succeeds, all actions succeed; if one action fails, all actions fail.

1. Splicing-In
* Mainnet Locking Transaction: Locks assets into the channel address
* Ascending Transaction: Ascends assets from the mainnet to the SatoshiNet, allowing them to circulate on the SatoshiNet
* Updates the Lightning channel status, increases channel capacity, credits the transferred assets to the initiator's name, and updates the Commitment Transaction

2. Splicing-Out
* SatoshiNet Destruction Transaction: Transfers a portion of the channel's assets to an OP_RETURN address on SatoshiNet, effectively destroying those assets
* Mainnet Unlocking Transaction: Withdraws a portion of the channel's assets to a designated address on the mainnet
* Updates the Lightning channel status, deducts the corresponding assets from the initiator's name, reduces channel capacity, and updates the Commitment Transaction