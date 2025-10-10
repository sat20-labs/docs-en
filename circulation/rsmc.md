RSMC
====

In the Lightning Network, a Revocable Sequence Maturity Contract (RSMC) is a type of contract used to ensure the security and reliability of Lightning Network channels.
RSMC is a time-locked contract that allows participants to revoke or close a Lightning Network channel under specific conditions. It is designed to prevent fraud and malicious activity and ensure transaction security.
Users' assets are locked in the channel. They maintain the security of their assets by holding commitment transactions and being able to construct penalty transactions. Users can withdraw their funds at any time without third-party permission.

## 1. Basic Principles of RSMC

Funds within a Lightning Network channel are not directly stored in "accounts." Instead, a pair of commitment transactions represents the current channel state.

* Each channel participant (Alice, Bob) has their own commitment transaction, which can be unilaterally settled on the main chain.
* The commitment transaction spends the output of the initial Funding Transaction.

**Funding Transaction**:

* Input: Contributions from both parties
* Output: A 2-of-2 multisig (<Alice, Bob>)
* This output can only be spent if both parties sign it.

**RSMC Mechanism:**

* Each commitment transaction contains a special "delayed spending" output with a revocation condition.
* This allows the other party to immediately spend the funds as a penalty if they broadcast an old commitment transaction (cheating).

The core RSMC output structure is as follows (using Alice's commitment as an example):

| Output Type | Description |
| ---------------- | -------------------------- |
| **to_local** | Payment to Alice, but with a delay (e.g., `OP_CHECKSEQUENCEVERIFY 144`) to prevent Alice from spending immediately. Contains a "revocation key" so that Bob can immediately spend the funds if Alice cheats and broadcasts the old transaction. |
| **to_remote** | Immediate payment to Bob (normal P2WPKH output) |

---

## 2. Channel Initial State: Alice & Bob Establish a Channel

Assumptions:

* Alice and Bob want to establish a 1 BTC channel.
* Alice contributes 1 BTC, Bob contributes 0 BTC.

### 1️⃣ Funding Transaction

* Output: `2-of-2 multisig(Alice_pub, Bob_pub)`
* Output Amount: 1 BTC
* TxID is assumed to be `funding_txid`

### 2️⃣ Both parties generate commitment transactions (C1a, C1b)

**C1a**: Alice's perspective (she signs and retains Bob's signature)
**C1b**: Bob's perspective (he signs and retains Alice's signature)

Assume the initial channel allocation is:

* Alice: 1.00000000 BTC
* Bob: 0.00000000 BTC

Then Alice's commitment transaction (C1a) is as follows:

| Output | Amount (BTC) | Condition |
| -------- | ---------- | ------------------------------------- |
| to_local | 1.00000000 | RSMC script (Alice can spend after a 144-block delay; otherwise, Bob can spend immediately using the revocation key) |

Bob's commitment transaction (C1b) is as follows:

| Output | Amount (BTC) | Condition |
| --------- | ---------- | ------------- |
| to_remote | 1.00000000 | Normal output paid to Alice |

At this point, both parties have signed each other's commitment transactions, but neither is broadcast.

Only the `Funding TX` is broadcast and confirmed.

--

## 3. Channel State Update

Suppose Alice wants to pay 0.3 BTC to Bob.

Target State:

* Alice: 0.7 BTC
* Bob: 0.3 BTC

A new commitment transaction pair is generated at this time:

* Alice's perspective: C2a
* Bob's perspective: C2b

### 1️⃣ Constructing a New Commitment Transaction

Alice's Commitment Transaction (C2a):

| Output | Amount (BTC) | Conditions |
| --------- | -------- | ---------------- |
| to_local | 0.7 | RSMC (Alice's Delayed Output) |
| to_remote | 0.3 | Normal Output, Immediately Given to Bob |

Bob's Commitment Transaction (C2b):

| Output | Amount (BTC) | Conditions |
| --------- | -------- | -------------- |
| to_local | 0.3 | RSMC (Bob's Delayed Output) |
| to_remote | 0.7 | Normal Output, Immediately Given to Alice |

### 2️⃣ Generate a New Revocation Key

* Each time a lightning channel is updated, both parties generate a new **revocation secret**.

* Each old state reveals the secret of the previous state to prevent cheating.

For example:

* Each state has a unique hash key pair `(revocation_secret_n, revocation_hash_n)`.

* If Alice and Bob update from C1 to C2:

* Alice sends Bob `revocation_secret_1` for her old state (C1).
* This way, if Bob detects Alice broadcasting C1 again, he can use this secret to spend Alice's delayed output.

### 3️⃣ Safely Replacing the Old State

When updating to C2:

1. Both parties sign the new commitment transaction (C2a, C2b)
2. Signatures are exchanged and verified successfully
3. Both parties reveal the revocation key for the previous state (C1)
4. The old state (C1) becomes invalid

At this point, if Alice broadcasts C1 again, she will be punished by Bob.

---

## IV. Penalty Mechanism: Alice Cheats by Broadcasting the Old Commitment Transaction (C1a)

Suppose Alice maliciously broadcasts the old commitment transaction **C1a** (her balance is 1 BTC).

Bob's node monitors on-chain transactions and discovers that someone has broadcasted the commitment transaction for this expired state (identified by TXID matching and script).

### 1️⃣ Bob Detects the Cheating

* Bob obtained `revocation_secret_1` during the previous update.
* He can immediately spend Alice's `to_local` output (i.e., 1 BTC).

The general structure of an RSMC script is as follows (simplified version):

```  
OP_IF  
    # If the other party provides the revocation secret, spend immediately.  
    <revocation_pubkey>  
OP_ELSE  
    # Otherwise, wait for the time lock before spending.  
    <delay> OP_CSV OP_DROP <local_delayed_pubkey>  
OP_ENDIF  
OP_CHECKSIG  
```  

### 2️⃣ Penalty Transaction

Bob constructs a "Penalty Transaction":

* Input: Alice's `to_local` output (the locked RSMC output)
* Signed with the public key corresponding to `revocation_secret_1`
* Output: Full 1 BTC to himself

This transaction does not require a time lock and can be spent immediately.

Final Result:

* Bob receives 1 BTC
* Alice loses all channel funds

---

## V. Complete Example Process Summary

| Stages | Events | Alice | Bob | Notes |
| -- | ---------- | ------------ | -------------- | ------------ |
| ① | Channel Opening | Contribute 1 BTC | Contribute 0 BTC | FundingTX on-chain |
| ② | Initial State | C1a: to_local=1 | C1b: to_remote=1 | Both Parties Have Commitments |
| ③ | Updated State (Alice → Bob 0.3 BTC) | C2a: to_local=0.7, to_remote=0.3 | C2b: to_local=0.3, to_remote=0.7 | C1's revocation secret is leaked after the update |
| ④ | Alice Cheats by Broadcasting C1a | On-chain Detection | Old State Detected | Bob obtains the revocation secret for Alice's old state |
| ⑤ | Bob initiates a penalty transaction | All funds (1 BTC) | Belong to Bob | Alice is penalized |

---

## VI. Watchtower
In actual use cases, Alice or Bob, as user nodes, cannot be online all the time. Therefore, a monitoring node is needed to monitor whether outdated commitment transactions are being broadcast. If so, it broadcasts the corresponding penalty transaction, helping the user punish the other party and recover all assets in the channel. This monitoring node is called a watchtower.