# Tumblebit Improvement Proposal (TIP)


## TIP 1


### Background


While implementing a Tumblebit server in “classic tumbler” mode and after carefully reviewing **Fig 3 of the original white paper describing interactions between Tumbler and Bob**, I came to the conclusion that steps 2,3,4 and steps 6,7 could be simplified (step 5 and 9-12 are unchanged).

The objectives are 
1. to stick to the most common cryptographic primitives (avoid Tumblebit-specific data formats) 
2. reduce the amount of data flowing back and forth between Tumbler and Bob.
3. preserve the security and privay-protection proprerties of the original Tumblebit protocol

Objective 1 is meant to facilitate integration of Tumblebit features into multiple wallet implementations: 
wallet developpers should be able to use their favorite Bitcoin library with minimal addtional code.

### Proposal


Initially, before each payment phase, Tumbler generates a fresh ECDSA key pair (PKT, SKT).

Bob generates 2 key pairs, “real” (PKB, SKB) and “fake” (PKF, SKF). Bob keeps PKF secret for now and publishes PKB.

In Fig. 3 **step 2**, Bob generates 15 “real” payout addresses (keeps them secret for now) and prepares 15 distinct “real” transactions.
In **step 3**, Bob prepares 285  “fake” transactions like so:

Fake transaction i pays Tumbler  compressed Bitcoin address (corresponding to PKT) 1 BTC (no network fee bearing in mind the transaction will never hit the blockchain ) with an OP_RETURN output containing the hex string “H || i” where H is the hash160 corresponding to the public key PKB.
Such fake transaction only sends a full refund to Tumbler and is unlikely to confirm without network fees.
_No need for Bob to generate (and later transmit to Tumbler) a set of 300 random pad values. Bob needs only to generate 2 regular, Bitcoin key pairs._

Bob hides the transactions in 300 sighash values (regular Bitcoin sighash computations here) , permutes them (**Step 4**), and finally sends them to Tumbler as beta1, ..., beta300.
_Minimized data flow: no need for Bob to send to Tumbler any hR, hF_ 

In Step 5, Tumbler signs each betai with SKT to create an ECDSA-Secp256k1 signature sigmai. 
_No change from original white paper._

**Step 6**: Bob sends to Tumbler the 15 “real” indices along with PKF
_Minimized data flow: Bob sends  a single public key PKF in lieu of salt value and 300 pad values._

**Step 7**: Tumbler can now compute the “fake” sighash values and verify that they match the “fake” betai values:
betai = sighash value of tx paying Tumbler 1 BTC with an OP_RETURN output bearing the hash160 digest corresponding to PKF. 

The hash160 is appended with i so that each i is a preimage of betai.
_No need for the CashOutTFormat nor the FakeFormat specified in the original white paper_

