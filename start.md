## On 51% attacks

(from chat on [2019-04-11](https://matrix.to/#/!NKtIRqGOEGaZvSQkKl:decred.org/$155495621013062aqmFL:decred.org))

degeri: (...) in PoS/PoW once someone 51% control they are in control forever. Is this argument sound?

davecgh: In a pure PoS with no other facts at play, yes. In Decred, or any other hybrid PoW/PoS that is setup to dilute influence, No.

All you have to do is think about how many tickets you could buy with your coins a year ago versus now. Assuming you were staking the same amount of coins then as now, your influence is most definitely less now than it was then, so even if you, perhaps, had 51% a year ago, you definitely wouldn't now.

Furthermore, 51% stake alone is not enough to "be in control". Whoever made that argument needs to spend a little more time digging into the math behind it.

degeri: That would be me. What is the percentage for POS "total control" ? Ie.On average 3 out of 5 tickets in ticket pool are yours. So that would be 60% ?

davecgh: It's a curve that also depends on the PoW. The crux of it is that there isn't a single number. https://medium.com/decred/decreds-hybrid-protocol-a-superior-deterrent-to-majority-attacks-9421bf486292

Scroll down to proof. One key observation is "If an attacker has around 50% of the stake/tickets, they would also need 1 times (100%) the honest hashpower." Notice that at 51% is still near 100% hash power needed.

jz: Assuming you could buy 12K DR5 units from Bitmain (you can't) it would set you back around USD $16M.

Then you need to fight it out in two other markets, the DCR market and the ticket market which both have constrained liquidity.

50% of the tickets represents USD $54M of DCR, but if you had to buy 2M DCR in the open market I would expect it to cost many multiples of that, and then you need to bid up the tickets, that's where the sdiff algo will send you back to buy more DCR in short order as ticket price ratchets up.

davecgh: I didn't calculate that 12K DR5, but if you're just taking the current hashpower and divdiing by the hash rate of a DR5, keep in mind that isn't correct because you are doubling the hash power in the process which means you only have 50%, not 100%.

Keeping the numbers easy, assume a global hash rate of 100PH/s. If you buy 100PH/s, there is now 200PH/s. So 100 / 200 = 50%.

jz: 12K DR5 is about equal to current total hash power I believe.

davecgh: Well, let's see. A DR5 is ~34TH/s.

```
$ ./dcrhashps.sh
From block difficulty: 460.41 PH/s
From last 120 blocks:  428.81 PH/s
```

We'll go with the lower value of ~429000 TH/s. 429000 / 34 ~= 12618. So yeah, that's what I figured.

degeri: So to summarize ... That argument only holds up to pure POS and not decred.

davecgh: Yes, it's true in pure PoS.

degeri: Someone needs both pow and POS super majority to keep others from taking back control .

jz: I don't see how you can prevent control from being regained. I can bid up hash and tickets too.

davecgh: Pure PoS with no other factors in play as well, since it's not impossible to devise a scheme such that there is some form of relative influence dilution there either, although it is much more difficult to do that securely, and, in fact, there is, as far as I know, no currently known way to solve the security proof for it.

The minority will eventually get a run of blocks that undoes everything the majority was trying to do at a rate of 4x.

jz: Right 20 tickets can be added and only 5 are coming out of the pool.

You would really need to dominate the PoW side to mitigate that effect.

davecgh: Assuming you could even buy the ~12618 DR5s, doing so would double the hash rate from the current ~429k TH/s to ~858k TH/s, means that 429k TH/s you just bought puts you at 50%. Same as the math above showed because it's a percentage.

What if you bought, 2x the current hash rate? 429k * 3 = 1287k. Then 858k / 1287k ~= 66%. See the formula? It's `n / (n+1)` where n is the ratio of the total hashpower being purchased. e.g. n=1 is 1x the current hashpower. n=2 is 2x the current hashpower, etc. Now, we can solve for something high like say, 90% and find that you'd need to buy 9x the current hash rate to end up with 90%.

(also see https://github.com/decred/dcrdata/issues/1022)

## Learning to code

(from chat on [2019-02-01](https://matrix.to/#/!HEeJkbPRpAqgAwhXWO:decred.org/$154901997725062UzBvj:decred.org))

[Go by Example](https://gobyexample.com/), [Effective Go](https://golang.org/doc/effective_go.html), and digging into existing quality code such as that in dcrd is about all you need to get up to speed on Go. That is probably actually Go's greatest strength. It is a super straight-foward language.

Unfortunately, that can also be its weakness when you're a much more experienced developer because it doesn't offer a lot more advanced things that make life easier such as immutability by default, zero-cost abstractions, and well-supported functional programming primitives. It's fantastic for readability though, and that is super important in a project like this.

## Mining

(from chat on [2019-03-28](https://matrix.to/#/!NNzHoaSdnsbZDQOXJr:decred.org/$155380788346482zglLE:decred.org))

There is only a single round of blake256 hashing required in Decred versus 2 in sha256 because blake does not have the vulnerabilities that require the double hashing. Then there is the fact that the Decred header provides ample nonce space so it's not necessary to completely recalculate the merkle root every 2^32 nonces (which an ASIC blows through in a few microseconds). The combined result is that it takes much less energy to find a solution for a given hash rate.

## Signatures

(from chat on [2019-02-05](https://matrix.to/#/!dhHYPTtCtvPSUfTepT:decred.org/$154933621028906vRveM:decred.org))

Signature hash calculation for transaction signing: https://github.com/decred/dcrd/blob/59ed4247a1d5816070852a332dcddff9322b9722/txscript/sighash.go#L225-L448

It uses blake256r14. The signature hash is then signed. The most common (and thus compatible with Bitcoin) type is indeed secp256k1 with ECDSA. The address is blake256r14+ripemd160+base58, but has a prefix of 2 bytes as compared to BTC's one byte. Here are the intermediate preimages to help with address costruction:

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

## OP Codes

its roughly the same as bitcoin script https://en.bitcoin.it/wiki/Script

You can see a map of all of the opcodes to their handlers here: https://github.com/decred/dcrd/blob/a1630f5af99334b64a09ff57476bf6afc801c3e6/txscript/opcode.go#L314-L599
Every handler is commented with the semantics including the data stack transformation(s).  You can click on each handler and go to its definition there on GitHub. 

## PoS Voting

PoS voting needs to be a pseudorandom process to prevent gaming.
So it becomes a question of how long the pseudorandom process takes, both on average and in a worst case.
We figured an average of 1 month was about right - ~30 days with the ticket and vote maturities
and we set the worst case time to ~4.7 months b/c we wanted to ensure that the vast majority of tickets would vote before expiring.
By keeping these lockup times long, but not too long, we filter out short term rent seekers which aligns the incentives of the stakeholders with the network.
jy-p

split tx is specific to ticket purchasing. when you buy a ticket, it requires a specific amount of dcr, so ticketbuyer creates txs with one or more outputs of the amount needed to buy a ticket
this was done to avoid the complexity of tracking change from ticket purchases since ticket purchases have an expiration and don't get mined with some regularity
e.g. issue 10 sstxs close to end of sdiff interval and there's a chance some of them won't get mined

## Lottery

(from reddit on [2019-04-02](https://www.reddit.com/r/decred/comments/b8k2i0/where_can_i_read_about_the_pseudo_random_number/))

https://github.com/decred/dcrd/blob/master/blockchain/stake/lottery.go

https://github.com/decred/dcrd/blob/64fee5100703ad5824bd9d85630710654c4d1d15/blockchain/blockindex.go#L256

https://www.reddit.com/r/decred/comments/b8k2i0/where_can_i_read_about_the_pseudo_random_number/

It uses a deterministic PRNG based on blake256 hashes that is seeded with the serialization of the header of the block that is being voted on suffixed with a constant derived from the hex representation of Pi which acts as a publicly verifiable NUMS number. From there, all of the eligible tickets (also referred to as live tickets) are sorted lexicographically by their hash to generate a total order. Finally, uniformly random values are produced (which obviously means it uses the total number number of eligible tickets as an upper bound to be able to properly produce them) and used as indices into the total order.

If you're comfortable looking at code, [this](https://github.com/decred/dcrd/blob/e9b2b4854f6e9f70907477a4cf689c1bcfbab327/blockchain/chaingen/generator.go#L1092-L1224) test harness code is pretty well documented and makes it clear what's going on. The aforementioned NUMS constant is defined [here](https://github.com/decred/dcrd/blob/e9b2b4854f6e9f70907477a4cf689c1bcfbab327/blockchain/chaingen/generator.go#L25-L29).

matheusd covered it in a video as well: https://www.youtube.com/watch?v=eysGWVhDFWY



## Websockets

(from chat on [2019-03-13](https://matrix.to/#/!HEeJkbPRpAqgAwhXWO:decred.org/$15524874501636tvwxN:zettaport.com))

https://github.com/jrick/wsrpc may be useful for working with the dcrd or dcrwallet JSON-RPC servers. It has a CLI tool to call RPCs, but with some differences from dcrctl. It's not Decred aware, and that allows you to make calls with arbitrary methods and args. Also supports websockets.

## Premine

(from chat on [2019-03-31](https://matrix.to/#/!lbzTjhzNbIaDbuAxkS:decred.org/$15540662511937zsfnk:decred.org))

jy-p:

The justification for the airdrop went like this: it took several years of software dev FTE to get Decred to the point it could be launched, which had a real fiat cost. CS/C0 had spent quite a bit on btcsuite prior to Decred's launch, which I did not think was fair to attempt to recoup in the initial premine. The total amount spent to do bringup on Decred was roughly USD 300K from C0, and devs earned USD 115K for sweat equity and direct buy in. We wanted to premine as little as possible to avoid giving the dev premine too many coins.

We put the figure for a maximum of the premine at 10% (2.1M DCR) and worked backwards from that. We wanted for the dev/airdrop split to be more like 40/60, but the staking system dictated that we needed to 50/50 split to ensure nobody could kill the chain early on.

Further, some of that 4% that went to the dev premine had to be staked to be certain nobody could easily attack the chain.

We were able to cut it from 10% to 8% by estimating a value of USD 0.49 per DCR.

Bitcoin is special b/c nobody knew that BTC was going to get big back then, so there is an effective premine of 1M btc. Whether it ever gets spent - who knows.

With PoS you need an "initial set of stakers" as you cannot stake with 0 coins. If miners=stakers it wouldn't create diversity among stakeholders. Hence the airdrop to the community/contributors to create the initial group of stakers. This made it possible to start with a multi-stakeholder situation.

Trying to compare PoW and PoW/PoS launches is hard b/c no PoS component changes the consensus algorithm in a major way. With PoS, if someone buys up all the coins and just doesn't participate in the pos system, the chain could die.

Airdrop created interest amongst a lot of ppl. ~2900 ppl registered. If you got your 282 DCR airdrop and just staked it all continuously, you'd have 1500+ DCR right now.

In many cases, we have worked backwards from a deliverable set we want to have to figure out how we should do it, versus 'forward' engineering everything. e.g. we collectively noticed that consensus rule changes were the most contentious changes to make in a cc project, so we then worked backwards from that to determine how we should go about making those changes. This is covered pretty extensively on the forum and the old bitcointalk thread.

People had to confirm their emails, we vetted the list manually for fraud and various games. There were some pretty creative games played by malicious actors.

chat 10-7-2019, jy-p
We wanted it 10% or under, we figured out that the best skew towards the airdrop we could do while ensuring the network couldn't be killed was 50/50, we needed to have a reasonable exchange rate for the dev premine coins. 8% of 21M gave 1.68M, 50% of that was 840K, total cost to do bringup to launch + opt-in from individual devs was USD 415K giving roughly DCR/USD of roughly 0.49, which we felt was fair considering the markets at the time.


Premine code:

https://github.com/decred/dcrd/blob/master/chaincfg/params.go#L478-L481

https://github.com/decred/dcrd/blob/47ade78c1a4e350313e9b5d9f5556f707ad22d0d/chaincfg/premine.go#L10

## Scaling

DCR General, 21 Apr 2019

elian:

How many transactions per second does decred can handle?
"Decred mainnet has blocksize 384 KiB. Let's take a rough average tx size of 400 bytes and average block time of 5 min: 393216/400/5/60 = 3.2 transaction per second."

haon:
With LN in place, it will be nearly infinite tx per second. Decred is open to scaling in different ways (not just off-chain) however, it's not needed (yet)

davecgh:

The real answer is that Decred is effectively infinitely scalable due to the fact the consensus rules have a solid governance process in place that allows them to be changed as necessary.  The constitution places no artificial limitation on them which says they must be set in stone and can never change.  You might choose to question that answer, but it really is the correct one.  Anything that is thought to be perfectly scalable today might very well not actually be scalable tomorrow.  The real key to scalability is flexibility. There is no guarantee that the current internet infrastructure will even look anything like it does today 50 years from now.  An entirely new technology with vastly different properties could replace it. Anyone claiming that their _current rules_ are perfectly scalable are either naive or outright lying to you, because they can't possibly know what the future holds.  It must be possible to adapt the system as technology changes to have any chance of that being true.


## Signature Hash
Decred was desinged to not require segwit for the things that bitcoin needs it for.
The "separate tx hashes depending on purpose" is how that is accomplished.
https://github.com/decred/dcrd/blob/a68540fc9aff9a38fa62c05bae680e00ad70e0da/txscript/sighash.go#L225-L441


## Block production times

https://www.reddit.com/r/decred/comments/8dszhf/expected_versus_actual_block_production_times/

davecgh | It's a Poisson process, so it's simply the PDF (technically PMF).  Further because it's the special case of one time in an interval, you can model it with the CDF for an exponential distribution.
Wolfram alpha link: https://www.wolframalpha.com/input/?i=cdf+exponential+distribution+%CE%BB%3D1%2F300
If you want to understand where all that comes from, see https://en.wikipedia.org/wiki/Poisson_distribution (note that lamba=1 and k=0 for block production), https://en.wikipedia.org/wiki/Probability_mass_function,
         | https://en.wikipedia.org/wiki/Exponential_distribution, and https://en.wikipedia.org/wiki/Cumulative_distribution_function.

## TX Fee calculations

https://github.com/decred/dcrwallet/blob/master/wallet/txrules/rules.go#L71
it calculates a fee based on the estimated size of the transaction once it is signed
https://github.com/decred/dcrwallet/blob/master/wallet/internal/txsizes/size.go

## HD Keychain
So you'll want to look at https://godoc.org/github.com/decred/dcrd/hdkeychain, and the example https://godoc.org/github.com/decred/dcrd/hdkeychain#example-package--DefaultWalletLayout  for how it's deriving children.  The difference is you'll just use https://godoc.org/github.com/decred/dcrd/hdkeychain#NewKeyFromString on the public extended key for the account and derive the immediate children for it.
Once you have a child, you turn it into an address, which is also shown in the example doco I linked.
'dcrctl getmasterpubkey default'
'dcrctl --wallet walletinfo | jq .cointype'

https://github.com/decred/dcrwallet/blob/52c3adc96d9b38200f4ca5f5d49eff0395e7d20c/wallet/udb/addressmanager.go#L63

https://github.com/decred/dcrwallet/issues/112

address derivation examples here: https://github.com/dcrlabs/hdaddy/blob/master/derivation.go
As far as the derivation path, just reference bip44 https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki
jrick's tool let's your type in the path. https://github.com/jrick/hdkey

## Links

https://faucet.decred.org/ usually can send you some testnet coins.

https://docs.decred.org/wallets/cli/dcrctl-basics/

https://testnet.dcrdata.org testnet explorer

#### RPC docs

https://github.com/decred/dcrd/tree/master/rpcclient

https://docs.decred.org/wallets/cli/dcrctl-rpc-commands/

https://github.com/decred/dcrwallet/tree/master/rpc/documentation

https://github.com/decred/dcrwallet/blob/master/rpc/documentation/api.md

https://github.com/decred/dcrdata#apis

#### reference implementations for Pythons, Node.js, C#, etc

https://github.com/decred/dcrwallet/blob/master/rpc/documentation/clientusage.md

#### Decred Change Proposals (DCPs)

https://github.com/decred/dcps

