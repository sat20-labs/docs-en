ORDX Protocol v2.0 (Not implemented)
====

The SAT20 asset issuance protocol, ORDX, will continue to evolve based on the needs of the BTC ecosystem while maintaining compatibility with previous versions.

This upgrade primarily aims to enhance the circulation of Satoshi assets on Layer 2 networks, while making the protocol more streamlined and user-friendly.


#### Data Writing Method
---
The data writing method in the ORDX protocol has been upgraded from the original inscription method to the OP_RETURN data method.

#### New Instructions
---
Version 2.0 primarily supports the destruction and swapping instructions for Ordinals NFTs.  
These two instructions have permanent effects and are initiated by the owner to either permanently destroy the corresponding Ordinals NFT or transfer it to another Satoshi.

##### Data Format:
```
OP_RETURN | MAGIC_NUMBER | CT_TYPE | CONTENT
MAGIC_NUMBER = OP_16
CT_DESTROY = OP_DATA_40  
CT_SWAP    = OP_DATA_41 
```

##### Destroy Content:
```
Satoshi | Inscription Number
```

##### Swap Content:
```
assetName | (start, end) | (start, end)
```
Note: The range of the input Satoshis must be controllable by the user. However, the output Satoshis are not necessarily under the user's control, meaning remote swaps are possible, allowing the binding asset of the Satoshis to be sent to another party.  
If the UTXO contains multiple ranges for the asset, each range requires a separate OP_RETURN execution.  
The format for assetName is: protocol:type:tickername (For NFTs, use inscription_number).


### Notes:
Please note that the above new features are still in the exploratory phase and have not yet been officially implemented.