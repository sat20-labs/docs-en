Satoshi Transcending Protocol (STP)
====

The core of the STP is the Lightning Channel, and the main content of the protocol is to coordinate the flow of Satoshi assets between the Layer 1 and Layer 2 networks. The flow of assets essentially refers to the flow of asset ownership, as Satoshi and assets are locked on the mainnet, but the ownership is transferred to Layer 2 for circulation, ultimately needing to return to Layer 1 for final settlement.

The principle of the protocol is that the user is the initiator. Almost all actions must be initiated by the user, while the remote service node acts as an automated responder that complies with the protocol and cooperates with the user's actions. This principle ensures that all operations reflect the user's intentions and maximizes the security of the user's funds.

Layer 1 is the mainnet, while Layer 2 can be any public chain supporting the Satoshi Transcending Protocol. In this example, we use the SatoshiNet.

---

### Opening a Channel
This is the standard method of opening a Lightning Network channel. The user provides the funds to open the channel, and the remote service node automatically responds and assists in opening the channel. Once the channel is opened, according to the Lightning channel's RSMC protocol, the user holds the commitment transaction. This is the fundamental guarantee for the user's fund security. Even if the remote node does not respond, the user can broadcast the commitment transaction to reclaim their funds.

Likewise, the remote service node also holds the commitment transaction. Both parties are on equal footing.

At this point, the user can perform other actions to allocate ownership of the funds in the channel or adjust the channel's capacity. These actions will trigger updates to the commitment transaction. Both parties only hold the latest commitment transaction. If one party broadcasts an old commitment transaction, the other party can construct a corresponding penalty transaction to clear the output of the old commitment transaction, effectively sweeping all the funds in the channel to their address. This is the power granted to users by the RSMC protocol.

Another important function of the Satoshi Transcending Protocol is that, when opening a channel, the Satoshi assets in the channel are automatically Ascended to the SatoshiNet and corresponding UTXOs are generated on the SatoshiNet. This means the assets have transcended to the Layer 2 network.

---

### Closing a Channel
The user can decide at any time to close the channel and reclaim their funds. There are two types of closure: negotiated closure and forced closure.

1. **Negotiated Closure**: Under normal circumstances, the remote node will be online and automatically support the user's closure request. The advantage of negotiated closure is that both parties can immediately spend their funds after the transaction is confirmed, with minimal fees.
   
2. **Forced Closure**: If the remote node is offline or permanently closed, the user can broadcast the latest commitment transaction to reclaim their funds. Unlike negotiated closure, the commitment transaction will first output the funds to an intermediate address, and the user will need to wait for a default CSVDelay (usually 144 blocks) before they can spend those outputs.

When executing the channel closure, the relevant assets on the Layer 2 network must perform a **Descend** operation, transferring the funds from Layer 2 back to Layer 1, ensuring that every Satoshi asset on the SatoshiNet is locked within the channel.

---

### Unlocking and Locking
After opening the channel, the assets automatically transcend to Layer 2 but remain locked in the channel. It can be imagined that Layer 1 and Layer 2 are two reservoirs, and the channel connects these two reservoirs. The funds locked in the channel are in a critical state and need special operations to move them in or out of either network. The operations for entering or exiting Layer 2 are unlocking and locking, while operations for entering or exiting Layer 1 are splicing.

**Unlocking and Locking** are operations performed on Layer 2:
1. **Unlocking**: Release the funds from the channel to the user's address on the SatoshiNet. Afterward, the user can freely sign and use their assets, just like on the mainnet.
   
2. **Locking**: Lock the funds back into the channel. Note that the channel's capacity is fixed, so the unlocking and locking operations cannot change the channel's capacity. Whether or not the funds can be locked back into the channel depends on the channel's capacity.

---

### Splicing
Splicing is a new protocol in the Lightning Network used to dynamically adjust channel capacity. However, the most widely used Lightning Network implementation, LND, has not yet implemented the splicing function.  
We use splicing technology to dynamically adjust the channel.

**Splicing** is an operation executed on Layer 1:
1. **Splicing-in**: Add a new UTXO to the Lightning channel, increasing the channel's capacity. This operation is equivalent to "recharging" the channel, allowing any asset to enter as long as the indexer supports it.
   
2. **Splicing-out**: Output funds from the Lightning channel by removing part of the UTXO to a specified address. This operation is equivalent to withdrawing assets from the channel.

Similarly, when performing splicing, since the effect is akin to depositing/withdrawing assets from Layer 2, the involved assets must automatically complete Ascend/Descend operations to ensure that the assets on Layer 2 match the assets in the channel.

---

The above outlines the core technology of the Satoshi Transcending Protocol, which is fully based on Lightning channels and represents another native extension protocol within the BTC ecosystem.