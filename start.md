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

Websockets

https://github.com/jrick/wsrpc
For working with the dcrd or dcrwallet json-rpc servers.
CLI tool to call rpcs, but with some differences from dcrctl.
It's not decred aware, and that allows you to make calls with arbitrary methods and args.
Also supports websockets.

Links

https://faucet.decred.org/ usually can send you some testnet coins.

https://docs.decred.org/wallets/cli/dcrctl-basics/

https://testnet.dcrdata.org testnet explorer

rpc docs

https://github.com/decred/dcrd/tree/master/rpcclient

https://docs.decred.org/wallets/cli/dcrctl-rpc-commands/

https://github.com/decred/dcrwallet/tree/master/rpc/documentation

https://github.com/decred/dcrwallet/blob/master/rpc/documentation/api.md

https://github.com/decred/dcrdata#apis

reference implementations for pythons, nodejs, c#, etc   

https://github.com/decred/dcrwallet/blob/master/rpc/documentation/clientusage.md


Premine

The justification for the airdrop went like this: it took several years of software dev FTE to get dcr to the point it could be launched, which had a real fiat cost. CS/C0 had spent quite a bit on btcsuite prior to dcr launch, which i did not think was fair to attempt to recoup in the initial premine. the total amount spent to do bringup on dcr was roughly USD 300K from C0, and devs earned USD 115K for sweat equity and direct buy in. we wanted to premine as little as possible to avoid giving the dev premine too many coins.

We put the figure for a maximum of the premine at 10% (2.1M dcr) and worked backwards from that. we wanted for the dev/airdrop split to be more like 40/60, but the staking system dictated that we needed to 50/50 split to ensure nobody could kill the chain early on

Further, some of that 4% that went to the dev premine had to be staked to be certain nobody could easily attack the chain

We were able to cut it from 10% to 8% by estimating a value of USD 0.49 per dcr
Bitcoin is special b/c nobody knew that btc was going to get big back then, so there is an effective premine of 1M btc. whether it ever gets spent - who knows.
  
With PoS you need an "initial set of stakers" as you cannot stake with 0 coins. If miners=stakers it wouldn't create diversity among stakeholders. Hence the airdrop to the community/contributors to create the initial group of stakers. This made it possible to start with a multi-stakeholder situation. 

Trying to compare pow and pow/pos launches is hard b/c no pos component changes the consensus algorithm in a major way with pos, if someone buys up all the coins and just doesn't participate in the pos system, the chain could die airdrop created interest amongst a lot of ppl. ~2900 ppl registered. If you got your 282 dcr airdrop and just staked it all continuously, you'd have 1500+ dcr right now

