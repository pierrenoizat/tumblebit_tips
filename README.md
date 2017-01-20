# Tumblebit Improvement Proposal (TIP)


## TIP 1


### Background


While implementing a demo Tumblebit server in “classic tumbler” mode and after carefully reviewing **Fig 4 of the [white paper](https://eprint.iacr.org/2016/575.pdf) describing interactions between Tumbler and Bob**, I came to the conclusion that steps 2,3,4 and steps 6,7 could be simplified (step 5 and 9-12 are unchanged).

The objectives are 

1. stick to the most common cryptographic primitives (avoid Tumblebit-specific data formats) 
2. reduce the amount of data flowing back and forth between Tumbler and Bob.
3. preserve the security and privay-protection proprerties of the original Tumblebit protocol

Objective 1 is meant to facilitate integration of Tumblebit features into multiple wallet implementations: 
wallet developpers should be able to use their favorite Bitcoin library with minimal additional code.


### Proposal


Initially, before each payment phase, Tumbler generates a fresh ECDSA key pair (PKT, SKT).

Bob generates an ECDSA key pair,(PKB, SKB), for "real" transactions and picks a single random 128-bit blinding factor r for “fake” transactions. Bob keeps r secret for now and publishes PKB.

In Fig. 4 **step 2**, Bob generates 15 “real” payout addresses (keeps them secret for now) and prepares 15 distinct “real” transactions.

In **step 3**, Bob prepares 285  “fake” transactions like so:

Fake transaction i pays Tumbler compressed Bitcoin address (corresponding to PKT) 1 BTC in output 0 (no network fee bearing in mind the transaction will never hit the blockchain ) with an OP_RETURN output (output 1) containing the hex string

`r || i `

Such fake transaction only sends a full refund to Tumbler and is unlikely to confirm without network fees.
_No need for Bob to generate (and later transmit to Tumbler) a set of 300 random pad values. Bob needs only to generate one regular, Bitcoin key pair and one random blinding factor._

Bob hides the transactions in 300 sighash values (regular Bitcoin sighash computations here) , permutes them (**Step 4**), and finally sends them to Tumbler as beta1, ..., beta300.

_Minimized data flow: no need for Bob to send to Tumbler any hR, hF_ 

In Step 5, Tumbler signs each betai with SKT to create an ECDSA-Secp256k1 signature sigmai. 
_No change from white paper._

**Step 6**: Bob sends to Tumbler the 15 “real” indices along with blinding factor r.

_Minimized data flow: Bob sends a single 128-bit blinding factor r in lieu of a salt value and 285 pad values._


| Original      | Size         | TIP 1        | Size        |
| ------------- |:------------:| ------------ |:-----------:|
| R, F          |  < 64 bytes  | R, F         | < 64 bytes  |
| hR, hF        | 64 bytes     |              |             |
| salt          | 16 bytes     |              |             |
| 285 random r  | 4560 bytes   | one random r | 16 bytes    |
| Total         |**4704 bytes**| Total        |**80 bytes** |


**Step 7**: Tumbler can now compute the “fake” sighash values and verify that they match the “fake” betai values:
betai = sighash value of tx paying Tumbler 1 BTC in output 0 with an OP_RETURN output (output 1) bearing the hex string 

`r || i `

The i index is appended to R so that each i is a preimage of betai.
_No need for the CashOutTFormat nor the FakeFormat specified in the original white paper_

**Conclusion**

With this proposal, 

1. the amount of data sent by Bob to Tumbler in steps 4 and 6 is reduced by 98% or 4624 bytes from the original protocol design.
2. the need for specific formats like CashOutTFormat and FakeFormat is avoided so a regular Bitcoin library is sufficient to build a wallet implementation.
