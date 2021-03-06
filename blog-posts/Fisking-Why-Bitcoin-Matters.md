
#Fisking "Why Bitcoin Matters"

In early 2014
[Marc Andreesen wrote a blog post][Andreesen-Why-Bitcoin-Matters]
arguing that Bitcoin is a pivotal technology for financial
transactions.  He made a number of expansive claims in the post, which
I feel are categorically false, and need to be debunked.  Hence, this
post.

In this post, I'll focus on two issues:  

* Bitcoin is not a solution to any distributed consensus problem
  worthy of the name -- and why this should matter to people who build
  distributed applications (like payment infrastructure)

* Bitcoin is _wildly_ more fraud- and crime-friendly than the current
  monetary system, and for good reason: its censorship-resistance is
  _intrinsically_ friendly to crime

## Bitcoin does not "solve Consensus"

>First, Bitcoin at its most fundamental level is a breakthrough in
 computer science ... Bitcoin is the first practical solution to a
 longstanding problem in computer science called the Byzantine
 Generals Problem.  More generally, the B.G.P. poses the question of
 how to establish trust between otherwise unrelated parties over an
 untrusted network like the Internet.

Andreesen here is referring to the "consensus" problem in distributed
systems, and specifically the version where computers can act in
"corrupt" (Byzantine) ways.  I'll explain below (1) what this problem
is and why it's important; (2) why **Andreesen is flat-out wrong**.

### Consensus and Byzantine Generals: A Short Tutorial

Andreesen refers to the "consensus" problem:

    can multiple computers on a network achieve agreement on the
	contents of a replicated database application, and the evolution
	of those contents over time, in the face of specified sorts of
	failures/corruptions/errors?

so that (for example) each computer (perhaps run by an independent and
competing bank) can trust that the contents of the database accurately
reflect the state of some set of transactions amongst the
participating banks.  Classically, the problem comes in two varieties:  

1. The "crash-fault-tolerance" consensus problem: in the face of
   (some) computers crashing and perhaps later restarting, can we
   ensure that non-crashed computers achieve consensus as to the
   contents of some replicated database application?

2. The "byzantine fault-tolerance" consensus problem (colloquially
   referred to as
   ["The Byzantine Generals Problem"][Lamport-Byzantine-Generals])
   (or BGP): in the face of (some) computers either crashing, or
   behaving in (arbitrarily undefined) corrupt ways, can we ensure
   that non-crashed, non-corrupt computers achieve consensus?

And further, the problem can be modified by the sort of failures
expected of the **network**:

* **Synchronous networks** are those where any message can be delayed
  only by some bounded amount: every message is eventually delivered
  (hence, **completely unrealistic**)

* **Asynchronous networks** are those that can arbitrary delay,
  duplicate, reorder, or drop messages: these are realistic networks.

So (for instance) the "Synchronous BGP" is one where computers can act
in corrupt ways, but the network _never partitions_.  Whereas, in the
"Asynchronous BGP", the network _can_ partition.

### Bitcoin does not "solve Consensus"

The question of whether Bitcoin solves the BGP
[has been studied][Bitcoin-theory-byzantine-generals] by academics,
and in my review of the literature, I have found _no_ examples of
academics claiming that Bitcoin solves the "Asynchronous BGP".
Rather, they all study the (unrealistic) "Synchronous BGP".  We
shoudln't be surprised: academics like to write papers with positive
results (nobody wants to tell the world that
[Omega-3 fatty acids are useless][omega-3-samefacts]), and this is no
exception.

So let's be clear:

    No evidence exists that Bitcoin can ensure that computers achieve
    consensus in a world with asynchronous networks (that can
    partition for arbitrarily long periods of time).  No evidence
    whatsoever.

[Sirer][Sirer2016] recently argued that Bitcoin was "strongly
consistent", by which he means, roughly speaking, that it achieves
consensus.  His argument boils down to this:

    Certainly, the most-recent Omega (typically == 6) blocks of the
    blockchain can be unstable (change over time due to "forks"). But
    once a block has been "buried" by Omega blocks, the probability of
    that block changing is vanishingly small.

Sirer relies on a probabilistic argument by
[Nakamoto in his original bitcoin whitepaper][Nakamoto-bitcoin] that
the more blocks that follow a particular block in the chain, the lower
the probability that the Bitcoin network will revise that block
(choose a longer fork that invalidates that particular block).  But
these probabilistic arguments are _all_ made with the assumption that
network partitions are short-lived.  And this assumption is
[not borne out by real-world experience][Aphyr]:  

>Average partition duration ranged ... to over 8.2 hours for
 hardware-related failures (... 95th percentile .... 3.7 days ...).  

It doesn't take a lot of reflection to conclude that if a network
partitions, the bitcoin miners on _both sides_ will continue to
operate (and hence, will diverge in their idea of the blockchain's
contents).  If the partition lasts "long enough", eventually Bitcoin
users on each side will act as if their trans are immutable/committed,
and .... "hand over the keys to their Maserati".  On one side or the
other, they will find themselves horribaly mistaken.

### Why does this matter?

[On 12 March 2013, the Bitcoin network forked (due to a software bug) for over six hours][bitcoin-2013-fork].
Bitcoin developers and operators met on IRC and resolved the problem.

>Over the next few hours, nearly every major Bitcoin developer and
 mining pool operator joined the bitcoin-dev IRC channel .... most
 large businesses .... shut down

It's true that the fork was due to a software bug, and software bugs
can happen in _all_ systems.  But Bitcoin is, by design, able to fork,
whereas [Paxos][Lamport-Paxos] and other (actually) consistent
protocols are (again by design) unable to do so.  A fork is something
regrettable to be avoided, but, like malaria and Africa, intrinsic to
Bitcoin.  In a Paxos-based implementation, a fork is a critical
software defect, and enormous effort is expended to ensure that such
things _never_ occur.

## Bitcoin is a haven for crime and ransomware

The worst howler in the piece concerns fraud and crime, and it goes to
the heart of why Bitcoin is a terrible idea:

>Finally, I’d like to address the claim made by some critics that
>Bitcoin is a haven for bad behavior, for criminals and terrorists to
>transfer money anonymously with impunity. This is a myth, fostered
>mostly by sensationalistic press coverage and an incomplete
>understanding of the technology. Much like email, which is quite
>traceable, Bitcoin is pseudonymous, not anonymous. Further, every
>transaction in the Bitcoin network is tracked and logged forever in
>the Bitcoin blockchain, or permanent record, available for all to
>see. As a result, Bitcoin is considerably easier for law enforcement
>to trace than cash, gold or diamonds.

Even at the time, in 2014, there was widely-deployed ransomware
demanding payment in Bitcoin, per
([Varonis][Varonis-Brief-History-of-Ransomware]), and a vanishing
number of ransomware demanding payment via _any other payment
mechanism_.  The claim that, because Bitcoin's ledger is publicly
visible, criminal activity can be tracked-down, was manifestly false
then, and time has shown us just how false it remains.  Numerous
([reports][Cryptowall-Report]) have shown us that Bitcoin is a
thieves' paradise.

What's even more amazing, is that the above words were written a
_half-year after the Mt. Gox_ collapse.  If Bitcoin had _any_ of the
purported properties, one would think that they would have resulted in
the swift (or at least, near-complete) recovery of all stolen coins.
Instead, in the Mt. Gox collapse 200,000 of 850,000 BTC were lost.  By
contrast, in the [MF Global collapse]:[MF-Global-collapse] 93 percent
of all funds were recovered.  In the recent case of the
[Bangladesh Central Bank theft][Bangladesh-bank-theft], $21 million
out of $101 million has so far been recovered, and it is instructive
to reflect on _why_ that recovery has been possible:

    The financial system today is _strongly_ biased (not enough, as it
    turns out) towards "know your customer", so tracing illicit
    transfers of funds is at least possible.

The problem is when those funds are sent into jurisdictions with lax
laws and supervisory practices, like the Philippines.  Or Bitcoin.
1
[Lamport-Byzantine-Generals]: http://research.microsoft.com/en-us/um/people/lamport/pubs/byz.pdf

[Bitcoin-theory-byzantine-generals]: https://bitcointalk.org/index.php?topic=99631.0

[Andreesen-Why-Bitcoin-Matters]: https://a16z.com/2014/01/21/why-bitcoin-matters-2/

[Sirer2016]: http://hackingdistributed.com/2016/03/01/bitcoin-guarantees-strong-not-eventual-consistency/

[Varonis-Brief-History-of-Ransomware]: https://blog.varonis.com/a-brief-history-of-ransomware/

[Cryptowall-Report]: http://cyberthreatalliance.org/cryptowall-report.pdf

[MT-Gox-collapse]: https://en.wikipedia.org/wiki/Mt._Gox#Insolvency_and_shutdown

[MF-Global-collapse]: https://en.wikipedia.org/wiki/MF_Global

[Bangladesh-bank-theft]: http://www.bloomberg.com/news/articles/2016-03-09/the-1-billion-plot-to-rob-fed-accounts-leads-to-manila-casinos

[Nakamoto-bitcoin]: https://bitcoin.org/bitcoin.pdf

[Aphyr]: https://aphyr.com/posts/288-the-network-is-reliable

[omega-3-samefacts]: http://www.samefacts.com/2013/01/health-medicine/how-breakthrough-medical-findings-disappear/

[bitcoin-2013-fork]: https://bitcoinmagazine.com/articles/bitcoin-network-shaken-by-blockchain-fork-1363144448

[Lamport-Paxos]: http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf
