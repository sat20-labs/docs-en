Smart Contracts
====

Channel Contracts
---

Based on the Lightning Channel RSMC contract, we have further upgraded the Lightning Network RSMC contract framework into a framework for coordinating multi-party actions, which we call a channel contract.

A channel contract is a contract recognized, jointly executed, and maintained by multiple parties. It can be thought of as a contract consisting of source code signed by multiple parties, which executes the actions specified in that code. The contract is currently provided as a template. By embedding these contracts in SatoshiNet nodes, contract deployment, execution, and verification are distributed.

Channel contracts implement some of the functionality of smart contracts, but do not currently have the full functionality of the EVM.

Currently available contract types include:
* Transcend, an asset transfer contract that replaces Lightning Network channels. Anyone can interact with this contract, freely transferring assets between the mainnet and SatoshiNet (via a multi-sig wallet interacting with SatoshiNet).
* Launchpool, a launch pool contract. Asset tickers can be deployed on the mainnet, minted, and automatically transferred to SatoshiNet for secondary distribution. After a successful launch, a pool remains, supporting AMM trading.
* AMM, an AMM trading contract, is an AMM trading pool that allows for adding liquidity. It offers fast transactions, low fees, and a share of pool profits.
* LimitOrder, a limit order trading contract. For those who prefer the AMM trading model, you can place limit buy or sell orders and trade at your desired price.

Channel contracts are one of SatoshiNet's key innovations, offering unlimited potential. Support for WASM is a key pillar of SatoshiNet's future development, and their functionality and flexibility far exceed other smart contract implementations on the Bitcoin network.

Script contracts
---
Upgrading the existing Bitcoin scripting language to a Turing-complete one is a path that has been explored and promoted on the mainnet for many years. While it's undeniably a viable approach, the design philosophy of the Bitcoin network conflicts with the need for a Turing-complete scripting language on-chain.

Bitcoin Script was designed from the outset as a restricted, non-Turing-complete stack language. It's primarily used to express "conditions under which a coin can be spent," rather than to perform general computations.

A programming language is Turing-complete if and only if it can:
* Express arbitrary computational logic (conditionals, loops, recursion, etc.)
* Have the ability to maintain infinite state (e.g., loops that can run indefinitely)

However, the Bitcoin Script language lacks:
* Loop constructs such as while, for, and goto
* Arbitrary memory read and write capabilities
* Dynamic jumps and function calls
* Dynamic stack expansion (with a limited stack depth)

Therefore, Bitcoin Script is strictly limited to a "bounded computation" model. This design principle aims to:
* Prevent denial of service (DoS) attacks
Every node on the blockchain must verify the script.
If a script allows loops or recursion, it could lead to infinite or highly complex execution.
This could result in a single malicious transaction overwhelming a node's CPU and causing verification bottlenecks.
Non-Turing completeness, however, guarantees that all scripts are executed within a finite number of steps.

* Predictable Verification Cost
The verification cost of each Bitcoin transaction must be deterministic:
Transaction verification time = block propagation time
Block propagation time determines the stability of network consensus
Non-Turing completeness → Verification complexity has a fixed upper bound, making network consensus more secure.

* Simplified Security Audits
Turing-complete languages ​​are extremely difficult to fully verify security (Halting Problem).
The non-Turing completeness of Bitcoin scripts allows for static analysis of each branch to determine the execution path.

* Realistic Constraints on Consensus Consistency and Decentralization
Every node must run the same script interpreter.
Complex scripting languages ​​are more prone to:
Inconsistent execution results across different platforms (endianness, integer overflow)
Implementation differences (consensus forks)

Therefore, Bitcoin is intentionally non-Turing complete to ensure the security, stability, and scalability of the mainnet. This also reflects Bitcoin's layered design philosophy. The Bitcoin mainnet (L1) is solely responsible for:
* Final settlement
* State security and censorship resistance

Complex computational logic is implemented by L2 or sidechains. The Lightning Network is designed to be an extension of the BTC network.

As the above discussion demonstrates, Bitcoin Script's non-Turing completeness is a by-design issue and cannot change this consensus. Even the community's current fervent calls for activating the OP-CAT instruction are unlikely to alter this consensus. The BIP347 proposal for activating OP-CAT is broadly as follows:
* Limiting the maximum length of concatenated results (e.g., ≤ 520 bytes)
* Prohibiting recursive concatenation
* Not introducing jump instructions
* Not changing the stack depth rule

This allows for safe recovery of OP-CAT without introducing loops or recursion.

Thus, the conclusion is:
* Without violating the design philosophy of "maintaining the mainnet's non-Turing completeness," OP-CAT is considered a "functional enhancement within a safety boundary."
* Activation is likely in the medium to long term (2025–2027), but the Bitcoin mainnet will still not achieve Turing completeness.

Therefore, the possibility of Turing-complete smart contracts on the mainnet is essentially zero.

While SatoshiNet can do more, if a technology has no potential for application on the mainnet, its worthiness depends on whether it is the optimal solution. Based on current research, further enhancements to the instruction set are necessary to achieve a Turing-complete scripting language, but this is relatively complex and comes at a high development cost. The already implemented channel contract solution offers low development costs and greater flexibility, making it the optimal solution for implementing smart contracts on SatoshiNet.