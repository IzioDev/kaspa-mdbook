# BIP-143-like SigHashes for Kaspa

This document outlines a change to the Kaspa protocol similar to
Bitcoin's BIP-143 proposal.  
The protocol outlined here is very similar to Bitcoin's proposal, 
with slight variations due to different transaction structure in Kaspa,
and the use of Blake2b hash (as opposed to SHA256 in bitcoin).

For motivation and further details see the original BIP-143 proposal: 
https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki

## Specification

### SigHashTypes
The SigHashTypes are defined as follows:
```
	SigHashAll          SigHashType = 0b00000001
	SigHashNone         SigHashType = 0b00000010
	SigHashSingle       SigHashType = 0b00000100
	SigHashAnyOneCanPay SigHashType = 0b10000000
```

Note this is different from bitcoin where SigHashSingle has the value 0b00000011.
This was changed to make SigHashType a true bit-field.
In addition, SigHashType is always serialized as a single byte, whilst in bitcoin it is often serialized as 
a 4-byte uint, and only 1-byte in the signature itself.

### SigHash calculation

A new transaction digest algorithm is defined:
```
Blake2b of the serialization of:
    1. tx.Version (2-bytes unsigned little endian)
    2. previousOutputsHash (32-byte hash)
    3. sequencesHash (32-byte hash)
    4. sigOpCountsHash (32-byte has)
    5. txIn.PreviousOutpoint.TransactionID (32-byte hash)
    6. txIn.PreviousOutpoint.Index (4-bytes unsigned little endian)
    7. txIn.PreviousOutput.ScriptPubKeyVersion (2-bytes unsigned little endian)
    8. txIn.PreviousOutput.ScriptPubKey.length (8-bytes unsigned little endian)
    9. txIn.PreviousOutput.ScriptPubKey (serialized as script)
    10. txIn.PreviousOutput.Value (8-bytes unsigned little endian)
    11. txIn.Sequence (8-bytes unsigned little endian)
    12. txIn.SigOpCount (1-byte unsigned little endian)
    13. outputsHash (32-byte hash)
    14. tx.Locktime (8-bytes unsigned little endian)
    15. tx.SubnetworkID (20-byte hash)
    16. tx.Gas (8-bytes unsigned little endian)
    17. payloadHash (32-byte hash)
    18. SigHash type of the signature (1-byte unsigned little endian) 
        (Note: SigHash type is different from bitcoin where it's 4-bytes)
    
Where:
    tx - the transaction signed
    txIn - the transaction input signed
```

### Semantics

The semantics of the original sighash types remain unchanged, except the followings:
1. The way of serialization is changed;
2. All sighash types commit to the amount being spent by the signed input
3. SINGLE does not commit to the input index. When ANYONECANPAY is not set, 
   the semantics are unchanged since previousOutputsHash and 
   txIn.PreviousOutpoint.* together implictly commit to the input index. 
   When SINGLE is used with ANYONECANPAY, omission of the index commitment 
   allows permutation of the input-output pairs, as long as each pair 
   is located at an equivalent index

The semantics of most values are straightforward, except the following:

#### previousOutputsHash
* If ANYONECANPAY flag is set, then previousOutputsHash is
  a uint256 of 0x0000......0000
* Otherwise previousOutputsHash is the Blake2b hash of the
  serialization of all input outpoints in the following format:
  ```
    1. previousOutpoint.TransactionId
    2. previousOutpoint.Index
  ```

#### sigOpCountsHash
* If ANYONECANPAY flag is set, then previousOutputsHash is
  a uint256 of 0x0000......0000
* Otherwise, sigOpCountsHash is the Blake2b hash of the serialization 
  of the SigOpCount of all inputs
  

#### sequencesHash
* If ANYONECANPAY, SINGLE or NONE sighash type is set, then sequencesHash is
  a uint256 of 0x0000......0000
* Otherwise, sequencesHash is the Blake2b hash of the serialization 
  of the Sequence of all inputs
  
#### outputsHash
* If the sighashType SINGLE and the input index is larger or equal to the 
  number of inputs or the sighashType is NONE, then outputs hash is a uint256
  of 0x0000......0000
* If sighashType is SINGLE and the input index is smaller then the number 
  of outputs, then outputsHash is the Blake2b hash of the following format for 
  the output with the same index as the input.
* Otherwise, outputsHash is the Blake2b hash of the following format for 
  all outputs.
```
  1. Value
  2. ScriptPublicKey.Version
  3. ScriptPublicKey.Script
```  

#### payloadHash
* If the tx is a native transaction (a.k.a. SubnetworkID = 0x0000......0000),
  then payloadHash is a uint256 of 0x0000......000
* Otherwise payloadHash is the Blake2b hash of the transaction's Payload