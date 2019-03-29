Learning to Code

https://gobyexample.com/
That, https://golang.org/doc/effective_go.html, and digging into existing quality code such as that in dcrd is about all you need to get up to speed on Go.
That is probably actually Go's greatest strength.  It is a super straight-foward language.
Unfortunately, that can also be its weakness when you're a much more experience developer because it doesn't offer a lot more advanced things that make life easier such as immutability by default,
zero-cost abstractions, and well-supported fp primitives.
It's fantastic for readability though, and that is super important in a project like this.


Mining

There is only a single round of blake256 hashing required in Decred versus 2 in sha256 because blake does not have the vulnerabilities that require the double hashing.  Then there is the fact that the Decred header provides ample nonce space so it's not necessary to completely recalculate the merkle root every 2^32 nonces (which an ASIC blows through in a few microseconds).  The combined result is that it takes much less energy to find a solution for a given hash rate.

Signatures

Signature hash calculation for transaction signing: https://github.com/decred/dcrd/blob/59ed4247a1d5816070852a332dcddff9322b9722/txscript/sighash.go#L225-L448   It uses blake256r14.  The signature hash is then signed.  The most common (and thus compatible with Bitcoin) type is indeed secp256k1 with ECDSA. The address is blake256r14+ripemd160+base58, but has a prefix of 2 bytes as compared to BTC's one byte.  Here are the intermediate preimages to help with address costruction:
```
privKey: 0000000000000000000000000000000000000000000000000000000000000001
 compressedPubKey: 0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798
ripemd160Preimage -- blake256(compressedPubKey): ba6f7ba0201fa407e7f6d48dc5d10f685c72b625f1d17442e986018d551836f6
pkHash -- ripemd160(ripemd160Preimage): e280cb6e66b96679aec288b1fbdbd4db08077a1b
checksumPreimage -- (prefix || pkHash): 073fe280cb6e66b96679aec288b1fbdbd4db08077a1b
checksum -- blake256(blake256(checksumPreimage))[:4]: d4de2763
base58Preimage -- (checksumPreimage || checksum): 073fe280cb6e66b96679aec288b1fbdbd4db08077a1bd4de2763
address -- base58(base58Preimage): DsmcYVbP1Nmag2H4AS17UTvmWXmGeA7nLDx
address: DsmcYVbP1Nmag2H4AS17UTvmWXmGeA7nLDx
```
The prefix for mainnet addresses based on secp256k1 with ECDSA is here: https://github.com/decred/dcrd/blob/59ed4247a1d5816070852a332dcddff9322b9722/chaincfg/mainnetparams.go#L211
