# EVM Developer Preview

This page is a testnet preview entry for Solidity / EVM developers. The SatoshiNet smart contract framework has been implemented and entered public testnet testing. EVM Runtime, template contracts, and Agent contracts are iterating around wallet, Explorer, Indexer, RPC, and developer tooling.

The current testnet has opened Agent / Prediction contract experience and deployed two standard EVM sample contracts: `ConstantProductAMM` and `LimitOrderBook`. EVM developers can use these samples to understand how Solidity contracts combine with the SatoshiNet UTXO asset model, ABI calldata, Result TX, and contract state root.

## What You Can Do Now

1. Enter testnet through [SAT20 PWA Wallet](https://sat20.org/pwa/?install=1).
2. Claim test GAS from PWA `Tools`.
3. Try Agent contract invocation in `Market -> Prediction`.
4. Try or deploy `ConstantProductAMM` and `LimitOrderBook` EVM samples from PWA `Tools -> Smart Contracts`.
5. Read the smart contract protocol, EVM rules, and asset precompile interface.
6. Prepare for the UTXO asset model differences needed when migrating Solidity contracts.

## What EVM Developers Need to Understand

1. EVM executes the Solidity / EVM state machine, but real SatoshiNet asset origin and settlement are expressed by the UTXO model.
2. ERC20 or application balances inside a contract are not the same as SatoshiNet native asset balances.
3. EVM invoke payload is Solidity ABI calldata. PWA can generate calldata from function signature and arguments, or accept raw calldata.
4. Asset and sats business inputs come from funding outputs transferred to the contract address; the same economic amount should not be duplicated in Solidity parameters. Test GAS / Result fee pays execution and result transaction fees.
5. To transfer SatoshiNet native assets, a contract must generate Asset Intents through the asset precompile and settle them through canonical Result TX.
6. EVM gas unit and the test GAS asset are not the same unit; current conversion is handled by protocol parameters.
7. Contract state participates in contract state root.
8. Deployment and invocation both require test GAS.
9. During testnet, tooling and RPC may change.

## PWA Interaction Model

When deploying an EVM contract, PWA reads the current compiler config from the node and uses fixed compilation settings to generate init code. Current test-stage defaults are `solc 0.8.30`, `evmVersion=paris`, optimizer `runs=200`, metadata `bytecodeHash=none`, and primarily single-file Solidity source. A valid Deploy TX must generate a canonical Result TX in the same block. Source, ABI, compiler config, and code hashes can be submitted as metadata to the L2 indexer for wallet, Explorer, and tool display.

When invoking an EVM contract, the basic PWA flow is:

1. Select contract address.
2. Enter function signature, arguments, sats, funding assets, and gas limit, or directly enter `calldataHex`.
3. Generate calldata and use the node estimate interface to check whether calldata, funding, and gas limit are acceptable.
4. Before signing, confirm that calldata still matches the current JSON parameters.
5. Sign and broadcast the Invoke TX.
6. If the execution produces asset transfer, refund, revert, or out of gas, check the same-block canonical Result TX.

Example invocation JSON:

```json
{
  "function": "swapAssetForSat(uint256)",
  "args": ["1000"],
  "sats": "0",
  "funding": [
    {
      "assetName": "brc20:f:ooxx",
      "amount": "100000"
    }
  ],
  "gasLimit": 5000000
}
```

Here `args` is only used for ABI encoding. `funding` is the real asset input transferred to the contract address in this invocation. The contract should use `fundingAssetAmount`, `fundingSats`, and `claimFundingAsset` to read and claim these inputs.

## Parameters Still to Publish

| Item | Content |
| --- | --- |
| RPC | Testnet EVM RPC / contract RPC entry |
| Chain ID | Testnet Chain ID |
| Example repo | Minimal Solidity contract, deployment script, and invocation script |
| CLI flow | install, compile, deploy, generate calldata, estimate, invoke |
| Explorer verification | Deploy TX, Invoke TX, Result TX, event, state root |
| Wallet interfaces | PWA / Wallet SDK deployment, invocation, calldata generation, and estimate interfaces |
| Common errors | GAS insufficient, RPC mismatch, wrong contract address, Result TX missing |

## References

- [Smart Contract Protocol](../protocol/contracts/readme.md)
- [EVM Contracts](../protocol/contracts/evm.md)
- [EVM Sample Contracts](evm-sample-contracts.md)
- [Smart Contracts and GAS](../learn/smart-contracts-and-gas.md)
- [Prediction Contract Quickstart](prediction-contract-quickstart.md)

**Page Status: In Development**
