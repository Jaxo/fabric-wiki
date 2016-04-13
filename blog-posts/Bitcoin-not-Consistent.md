
# Bitcoin Does Not Provide Strong Consistency

>The network can stay partitioned longer than you can wait before
 handing over the keys to your Maserati

In this article, I want to explore the question of exactly whether,
how, and why, we cannot trust Bitcoin with financial transactions, and
other economically-relevant databases.  In short, if you're selling
your Maserati for Bitcoin, you need to be an oracle -- to know that
the network has not partitioned, or if it has, to know by when it will
be repaired.  These are unknowable for technical folk, and beyond
human imagination for nontechnical folk.  The fact that to use
Bitcoin, we *must* know these things, means **Bitcoin is unsafe as the
underpinnings of our economy**.

Recently [Emin Gun Sirer][Sirer2016] argued that Bitcoin provides
strong consistency, not eventual consistency (as was argued by
[Bitcoin Meets Strong Consistency][DeckerSeidelWattenhofer2014]).  In
this short note, I want to do two things:  
* **provide a brief counter-example**.  It suffices to understand only
  the publicly known properties of Bitcoin, to prove that Bitcoin
  cannot offer anything that anybody would _recognize_ as "strong consistency".  
* **Briefly re-explain [_consensus_][Wikipedia-Consensus] ** and
  re-cap the counter-example, as a proof that Bitcoin cannot solve the
  consensus problem.

First though, why should any of this matter?  Bitcoin seems like a
simple payment model, requiring a trivial database (one four-column
table).  If, from time-to-time, an inconsistent set of transactions
turns up, due to a too-long partition, perhaps that's the price we pay
for a global-scale payment system.  The problem with this attitude, is
that _payment systems in the real world_ aren't trivial.  The
[history of payment systems][Nacamuli-Payment-Systems] tells us that
modern payment systems are complex for reasons having _nothing_ to do
with the underlying technology.  They must deal with (at a minimum):  
* **default risk**: (banks keep track of how much they're owed by other
  banks, and this forces those other banks to reorder their payments
  to minimize their balances)  
* **liquidity costs**: banks may need to borrow funds short-term to make
  payments (the basic function of banks, let's remember, is to borrow
  short and lend long)  

These issues directly correspond to complexity in the database
application (for example, queueing and reordering payments to minimize
both kinds of issues, prioritizing high-value customer traffic over
low-value customer traffic, etc).  Similarly, most retail brokerage
applications (another touted application of blockchains) have complex
data-models -- including details like tax-lot calculations, multiple
and complex access-control, etc, etc, etc.  We haven't even begun to
scratch the surface of "smart, self-enforcing contracts for bond
coupon or dividend payments".  Or "bond coupon payments on mezzanine
tranches of CDOs".  The complexity _should_ boggle the mind, and
convince one that we should expect arbitrary database applications,
not simplistic systems like Bitcoin.

And such database applications, almost by default, require strong
consistency.

## Back to Our Counter-example: Bitcoin is not Consistent

Sirer's argument is that Bitcoin provides "strong consistency".  By
this, he means that, even though at any point,  

>the last few blocks of a blockchain are subject to rearrangement  

there is some number of blocks Omega such that, if we discard those
last Omega, then the _rest_ of the chain is unchanging and
consistent.

>

From this, arises a recipe for "knowing if you got paid":  
1. look for your transaction in the blockchain  
2. wait until Omega blocks have appeared "above" your tran (in the
   parlance, your transaction has been "buried" by Omega blocks)  
3. look again for your transaction; if it still appears in that block,
   you can trust that you got paid.  
Since Bitcoin generates blocks more-or-less every ten minutes, this
corresponds directly to a waiting period.

So let's make this crystal-clear.  The argument is:

    If we can put an upper bound on how long we need to wait, for the
    block in which our tran is found to move from the scratchpad, to
    the committed part of the chain, then effectively Bitcoin (with
    that wait) is a consistent system.

This is false for one simple reason:

    Networks can and do partition, and for unpredictably long times.

If the bitcoin network partitions, then each partition (side of the
network) can and will proceed onwards, mining new blocks and extending
the blockchain.  This is touted as a _strength_ of Bitcoin (and is the
essence of "permissionlessness": in a sense, each partition acts as if
the other side has just "exited" the network).  When the network
partition is repaired, one or the other side will find that _all_ the
blocks mined since the partition have disappeared.

I want to make another thing crystal-clear:

    Sirer is right, that if you wait Omega = 6 blocks, the chance of
    seeing an inconsistency is vanishingly small.

Until a partition that lasts longer than an hour.  And as we all know,
[the network is reliable.][Aphyr], so that never happens.

## Consensus and why Bitcoin doesn't achieve it

Later in the post, Sirer asks

>Why Is This So Hard?

>There is a lot of confusion among software developers when it comes
>to consistency in databases. The muddled thinking that pervades the
>valley when it comes to issues of consistency probably stems from two
>sources: (1) academics' failure to provide a simple, accessible
>framework for reasoning about consistency, coupled with the
>misleading framework embodied in CAP

But it _isn't_ hard!  [Brewer's CAP theorem][Brewer-cap-theorem] isn't
needed to understand _consensus_ in a distributed system, and
consensus _directly_ corresponds to the properties we want from a
database.  So let's go thru the story of consensus.

Consensus is [defined by Lamport][Lamport-Paxos-Made-Simple] in terms
of three properties of a system that _chooses_ values:  
* **Integrity** Only a value that has been proposed, may be chosen.  
* **Agreement** Only a single value is chosen.  
* **Validity** A process never learns that a value has been chosen,
unless it actually has been.

In the context of a blockchain, we can think of consensus in terms of
the blocks: we achieve consensus on the value of each block,
one-by-one.  Each process participating in the system (each _miner_)
is choosing blocks, and to say that the protocol the miners execute,
achieves consensus, means precisely that the values chosen by the
miners satisfy the three properties above.  

    And in this presentation, we can assume that the meaning "choose
    the value of block N" is "choose the value of block N, afer
    Omega=6 blocks have buried block N"

It should be clear now that Bitcoin (in the face of arbitrary network
partitions) cannot achieve **Agreement**: on each side of the
partition, miners choose the value of a block, and if the partition
lasts long enough, that choice outlasts Omega .... so applications
that rely on this definition of "choose" _will_ act on the assumption
that the block is irrevocable, only to find (on one side of the
partition) that when the partition heals, the block is indeed revoked.

## Consensus is the Right Property

Consensus is the right property to ask of a distributed database
application.  **Integrity** means that the system does not invent
transactions in its own -- each transaction that is chosen, was
submitted by a client.  **Agreement** means that, when asking a node
for the contents of block NNN, _if it gives an answer_n, that answer
will be identical to the answer of every other answering node.  And
**Validity** means that a process will not conclude that block NNN has
value A, when in fact, other processes have concluded it has value B.

Taken together, these properties allow a client of the blockchain to
know that the sequence of blocks emitted by some miner to which it is
connected, is the same sequence received by any other client of any
other miner in the network.  Perhaps this client will receive blocks
with some delay; perhaps this client will stop receiving blocks for an
arbitrary period (if the network partitions, and this client is on the
side with fewer nodes) but it will _never_ receive a block that the
miner later revises.

From this guarantee, to building a distributed database application,
is now straightforward: we treat the blockchain as the (write-ahead)
log of a database and merely apply the transactions in that log to the
database.  We don't need to concern ourselves with how to revise the
database when forks occur, or any other "interesting" issues: if a
query to our (local) database gives us an answer, we can know that
(modulo the possibility that the answer is out-of-date) the answer is
the same as we'd get from any other replica.  And if we want an answer
that is guaranteed to not be out-of-date, we merely perform what is
known as a "strong read".

## Conclusion

It should be clear by now that Bitcoin does _not_ provide strong
consistency: (under partitions) Bitcoin nodes can diverge from each
other for arbitrarily long periods of time, and with no indication of
this to clients.  And by contrast, blockchains based on proper
consensus (e.g. [Tendermint][Tendermint], [Hyperledger][Hyperledger],
[Juno][Juno]) simply don't have these issues.

[Sirer2016]: http://hackingdistributed.com/2016/03/01/bitcoin-guarantees-strong-not-eventual-consistency/
[DeckerSeidelWattenhofer2014]: http://arxiv.org/pdf/1412.7935.pdf
[Wikipedia-Consensus]: https://en.wikipedia.org/wiki/Consensus_(computer_science)
[Aphyr]: https://aphyr.com/posts/288-the-network-is-reliable
[Lamport-Paxos-Made-Simple]: http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf
[Nacamuli-Payment-Systems]: http://www.amazon.com/Payment-Systems-Macmillan-Financial-Institutions/dp/0230202500
[Brewer-cap-theorem]: https://en.wikipedia.org/wiki/CAP_theorem
[Tendermint]: http://tendermint.com/
[Hyperledger]: https://www.hyperledger.org/
[Juno]: https://github.com/buckie/juno
