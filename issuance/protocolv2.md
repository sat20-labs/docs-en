ORDX Protocol v2.0
====

The SAT20 asset issuance protocol, ORDX, will continue to evolve based on the needs of the BTC ecosystem while maintaining compatibility with previous versions.

This upgrade mainly improves the circulation of sat-bound assets on Layer 2 while simplifying the protocol and making it easier to use.


Data Writing Method
---
The ORDX protocol upgrades its data writing method from inscriptions to OP_RETURN payloads.

New Instruction
---
Version 2.0 primarily introduces two new instruction groups:

1. Asset unbind
2. Address-level freeze / unfreeze

### 1. Asset Unbind

This instruction has a permanent effect.
Only the asset owner can initiate unbind. The target UTXO or UTXOs must be used as inputs in the same transaction, transferred to a specific `vout`, and accompanied by an `OP_RETURN` unbind instruction that specifies `ticker + vout`. All assets of the specified ticker in that output are unbound. Other asset types are not affected.

Data Format:
```
OP_RETURN | SAT20_MAGIC_NUMBER | CT_TYPE | CONTENT
SAT20_MAGIC_NUMBER = OP_16
CONTENT_TYPE_UNBIND = OP_DATA_40
CONTENT = target ticker name + vout
```

### 2. Address-Level Freeze / Unfreeze

This instruction is mainly intended for controlled-asset scenarios such as stablecoins.

The target of freeze and unfreeze is the asset permission state of a specific address under a specific ticker.

The core rules are:

1. Freeze is address-level, and the target object is `ticker + address + height`.
2. The freeze effective height is explicitly carried in the instruction content. The freeze instruction is valid only when `confirmation height - freeze height <= 2`.
3. The assets currently held by the frozen address under that ticker, as well as any future inflows of that ticker to the same address, are treated as frozen assets.
4. Frozen assets can no longer be used as valid transferable assets for that ticker.
5. If the underlying BTC UTXO is spent, the frozen assets do not transfer with the outputs and are instead destroyed at the protocol layer.
6. After unfreeze, UTXOs at that address that still carry that ticker can transfer normally again. Assets already destroyed at the protocol layer due to spending are not restored.

Initiation authority:

1. Only tickers with `selfmint=100` can be frozen or unfrozen.
2. Only the deployer of that ticker can initiate freeze and unfreeze.
3. Here, deployer means the current owner of the ticker's deploy inscription, not the original inscription address at deploy time.

Data Format:
```
OP_RETURN | SAT20_MAGIC_NUMBER | CT_TYPE | CONTENT
SAT20_MAGIC_NUMBER = OP_16
CONTENT_TYPE_FREEZE = OP_DATA_43
CONTENT_TYPE_UNFREEZE = OP_DATA_44
CONTENT = target ticker name + target address + freeze height
```

- `target ticker` must be an ORDX FT asset.
- `target address` is a standard address string.
- `freeze height` is required only for freeze. Unfreeze does not carry a height.
- Whether an address is currently frozen is determined by the most recent freeze or unfreeze instruction for that ticker.

This feature is activated in indexer `1.5.0`.
