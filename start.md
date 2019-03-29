Mining

There is only a single round of blake256 hashing required in Decred versus 2 in sha256 because blake does not have the vulnerabilities that require the double hashing.  Then there is the fact that the Decred header provides ample nonce space so it's not necessary to completely recalculate the merkle root every 2^32 nonces (which an ASIC blows through in a few microseconds).  The combined result is that it takes much less energy to find a solution for a given hash rate.
