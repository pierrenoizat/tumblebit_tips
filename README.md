# Tumblebit Improvement Proposal (TIP)


## TIP 1


### Background


While implementing a demo Tumblebit server in “classic tumbler” mode and after carefully reviewing **Fig 4 of the [white paper](https://eprint.iacr.org/2016/575.pdf) describing interactions between Tumbler and Bob**, I came to the conclusion that steps 2-7 could be simplified (steps 9-12 are unchanged).

The objectives are 

1. stick to the most common cryptographic primitives (avoid Tumblebit-specific data formats) 
2. reduce the amount of data flowing back and forth between Tumbler and Bob.
3. preserve the security and privay-protection proprerties of the original Tumblebit protocol

Objective 1 is meant to facilitate integration of Tumblebit features into multiple wallet implementations: 
wallet developpers should be able to use their favorite Bitcoin library with minimal additional code.


### Proposal


Initially, before each payment phase, Tumbler generates a fresh ECDSA key pair (PKT, SKT).

Bob generates an ECDSA key pair,(PKB, SKB), for "real" transactions and picks a single random 256-bit blinding factor r for “fake” transactions. Bob keeps r secret for now and publishes PKB.

In Fig. 4 **step 2**, Bob generates 42 “real” payout addresses (keeps them secret for now) and prepares 42 distinct “real” transactions.

In **step 3**, Bob prepares 42 “fake” transactions like so:

Fake transaction i pays Tumbler compressed Bitcoin address (corresponding to PKT) 1 BTC in output 0 (no network fee bearing in mind the transaction will never hit the blockchain ) with an OP_RETURN output (output 1) containing the hex string

`r || i `

Such fake transaction only sends a full refund to Tumbler and is unlikely to confirm without network fees.
_No need for Bob to generate (and later transmit to Tumbler) a set of 84 random pad values. Bob needs only to generate one regular, Bitcoin key pair and one random blinding factor._

Bob hides the transactions in 84 sighash values (regular Bitcoin sighash computations here) , permutes them (**Step 4**), and finally sends them to Tumbler as beta1, ..., beta84.

_Minimized data flow: no need for Bob to send to Tumbler any hR, hF_ 

In **Step 5**, Tumbler signs each betai with SKT to create an ECDSA-Secp256k1 signature sigmai. Tumbler picks 84 random 256-bit symetric encryption keys epsilon and computes ci = AES256( epsiloni, sigmai )
_Minimized data flow: we propose a simple AES256 encryption instead of the complex SHA512 hash specified in the original paper: this yields most of the data traffic reduction_ 
 

**Step 6**: Bob sends to Tumbler the 42 “real” indices along with blinding factor r.

_Minimized data flow: Bob sends a single 256-bit blinding factor r in lieu of a salt value and 42 pad values._


| Original         | Size          | TIP 1        | Size          |
| ---------------- |:-------------:| ------------ |:-------------:|
| 84 Betas         | 2688 Bytes    |              | 2688 Bytes    |
| hR,hF            | 64 Bytes      |              | 0             |
| 84 (z,c) couples | 26880 Bytes   |              | 24192 Bytes   |
| Salt             | 32 bytes      |              | 0             |
| 42 random r      | 1344 Bytes    | One random r | 32 Bytes      |
| 41 Quotients     | 10496 Bytes   |              | 10496 Bytes   |
| 42 Epsilons      | 10752 Bytes   |              | 1344 Bytes    |
| Total            |**52256 Bytes**|              |**38752 Bytes**|
| Reduction        |               |              |**26%**       |


**Step 7**: Tumbler can now compute the “fake” sighash values and verify that they match the “fake” betai values:
betai = sighash value of tx paying Tumbler 1 BTC in output 0 with an OP_RETURN output (output 1) bearing the hex string 

`r || i `

The i index is appended to r so that each i is a preimage of betai.
_No need for the CashOutTFormat nor the FakeFormat specified in the original white paper_

**Conclusion**

With this proposal, 

1. the total data flow in the interactions between Bob and Tumbler is reduced by 26% as a result of the simplification and use of AES256 for symetric encryption.
2. the need for specific formats like CashOutTFormat and FakeFormat is avoided so a regular Bitcoin library is sufficient to build a wallet implementation.
