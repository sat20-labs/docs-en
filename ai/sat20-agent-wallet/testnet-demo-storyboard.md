# One-Minute Testnet Demo Storyboard

This storyboard is the short public version of the SAT20 Agent Wallet / STP demo. It targets Bitcoin-native users who understand UTXOs and channel safety but do not want implementation internals.

Current English outputs:

- [Narrated one-minute demo](video/sat20-agent-wallet-one-minute-en.mp4)

## One-Minute Version

| Time | Visual | Message |
| --- | --- | --- |
| 0:00-0:06 | SAT20 PWA wallet asset and channel view | The user connects to a Core Node through a private STP channel. This is not a custodial account |
| 0:06-0:12 | Bitcoin testnet4 funding tx | BTC enters a 2-of-2 channel address; this is the L1 safety boundary |
| 0:12-0:24 | L1/L2 indexer and splicing tx | Runes, BRC20, and ORDX assets can enter or exit the channel through real transactions |
| 0:24-0:36 | SatoshiNet explorer and unlock/lock tx | Assets move on SatoshiNet and can return to channel protection |
| 0:36-0:48 | `stp.safety_snapshot` output | The Agent reads channel point, commit height, commitments, and punishment coverage |
| 0:48-0:56 | Old commitment and punish tx on Bitcoin testnet4 | The Core Node broadcasts an old state; the user wallet broadcasts punish transactions |
| 0:56-1:00 | PWA wallet and final safety caption | The user has chain exit and old-state punishment ability |

## Narration

```text
SAT20 uses STP to move Bitcoin-native assets into SatoshiNet without turning them into custodial balances.

First, the wallet opens a 2-of-2 channel with a Core Node. The funding transaction is visible on Bitcoin testnet4.

Then Runes, BRC20, and ORDX assets enter the channel through splicing and anchor evidence. On SatoshiNet, the wallet can unlock assets to a personal address and lock them back into channel protection.

Before every value movement, the Agent reads a safety snapshot: channel point, commitment height, latest commitments, and punishment coverage.

In the testnet drill, the Core Node broadcasts an old commitment. The wallet detects it and broadcasts punishment transactions.

The result: assets can flow through SatoshiNet, while the user keeps verifiable exit and punishment paths.
```

## Screen Requirements

1. Show real txids, shortened only in captions.
2. Hide mnemonics, private keys, passwords, and revocation secrets.
3. Mark the video as Bitcoin testnet4 / SatoshiNet testnet.
4. Use real PWA, explorers, indexer JSON, and skill output.
5. Do not show debugging, node logs, or temporary repair steps.

## Evidence Checklist

The short video needs:

1. One funding tx.
2. One Runes or BRC20 splicing tx.
3. One unlock tx.
4. One lock tx.
5. One safety snapshot.
6. One old commitment tx.
7. One punish tx or punish package.
8. Final channel safety or fresh-channel state.
