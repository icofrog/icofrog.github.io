Author: Matthew Roberts (matthew@roberts.pm)

Date: July 1, 2017

Whitepaper: [0x white paper v1.pdf](https://github.com/icofrog/icofrog.github.io/blob/master/0x%20Project/0x_white_paper_v1.pdf)

# 1. Introduction

The 0x Project is an attempt to introduce a decentralized exchange
protocol for ERC-20 tokens in the Ethereum world. The exchange focuses
on building a system that can be used to incentivize users to run
servers that will serve as order books to match orders within the
system.

# 2. Technology

The 0x Project defines a peer-to-peer network of order book servers
who are incentived to provide the service by receiving fees when a user
elects to use their provided match.

This design presents multiple issues.

## 2.1. Poor scalability

To progate open orders, the protocol specifies a broadcast algorithm
where peers submit an order across the entire network. This routing
algorithm is necessary to ensure that every order book server has
the same amount of liquidity but it means that the protocol cannot
scale to high transaction volumes.

## 2.2. Race conditions with matching

0x doesn't define enforcible matching. It is possible to submit an
order across the network that is filled by multiple people (partially
or in full) before a user has had chance to fill it themselves.

Since orders are filled asynchronously, order servers have no way to
stay up to date with matches except to watch transactions confirm on the
blockchain. This creates a race condition where users race to fill an
order, wasting resources across the network and consuming unnecessary
Ether as fees in the form of rejected on-chain transactions.

## 2.3. Matches are unenforcible

A standard limit-based order book works by sorting all the sell orders
from lowest to highest and all the buy orders from highest to lowest.
The sell orders can then be put next to the buy orders to tell traders
the bid-ask spread - a measure of the difference between the lowest
price a seller is willing to sell at and the highest price a buyer is
willing to buy at.

The bid-ask spread is useful because it can be used to ensure traders
get access to accurate pricing information, knowledge of liquidity, and
so fourth. But pricing information in 0x cannot be determined because
matching is unenforcible. A person may indicate to a matching
server they are after a certain trade but there is no part of the
protocol that determines whether they go through with it.

This means that pricing cannot be determined since all open orders in
the order book cannot be matched. Another consequence of this is that
traders can never be ensured that they will get the best possible
price for an order because they cannot be guaranteed to match against
the tip of the order book. 

## 2.4. Incentives ignore the cost of operation

0x defines an incentive system where matching servers are rewarded
a fee from a user if they use a match that it provides. The issue is
that for this to occur the matching server needs to store all broadcast
orders and it has no way to guarantee that a match that it offers up
will end up being used by a trader.

The consequence of this means that there is no way to ensure that a
match server in 0x will be paid for their service. The protocol does not
define any incentives for the cost of bandwidth, disk space, and so
fourth which provides a poor reason to run the software. 

The 0x token doesn't make sense in this context.

# 3. Suggested fixes

1. Use a single server for matching. To keep the system decentralized,
the server could be selected via a random lottery protocol. You could
define an incentive system that requires a bond to participate in
the lottery. Winning the lottery would entitle a user to act as a
matching server which is then used by everyone else in the network. 

2. Use 3 of 3 multi-sig in the virtual swap transactions. The three
signatures defined are the bidder, the asker, and the matcher.
The asker provides a signature to the match server to open the order.
The match server offers up a match to a bidder and locks the match
for N minutes. The bidder signs the match locking it in. The match
server provides the final signature and the bidder publishes the match
to the contract which checks whether the deposits are correct and
all signatures have been provided.

3. To ensure that a matching server has the resources necessary to
provide a high quality of service to the network, a system could be
devised to audit its bandwidth which could also be part of the lottery
protocol random selection. Based on these numbers, matching could be
shared between as many servers as necessary, who would each have a
guaranteed stake in the reward for providing matching. 

## 3.1. Decentralization

Decentralization is preserved because anyone can participate to be
an order book provider. Sybil attacks are mitgated by requiring a
large bond. For more information look at the design in the TruthCoin
white paper since it deals with similar problems.

## 3.2. Scalability

Scalability is improved because flooded orders are no longer required.
Orders are instead submitted directly to public servers that everyone
agrees to use.

## 3.3. Incentives

Because matching is now enforcible and a single matching server is
in control at any given time, the matching server is guaranteed
that it gets paid for providing its service. The chances of a single
matching server DoSing the network could be reduced by having a
redundant server with a shared reward system.

In a full DDoS, users could vote to change the server responsible for
matching. This random lottery protocol would define the validity of
the matching signatures within the exchange contract code.

## 3.4. Syncing orders between servers is efficient

Enforcible matching allows the order book to be sorted and organized
more efficiently, thereby guaranteeing traders get the best price with
accurate pricing information. The idea allows for efficient syncing:

Since orders are sorted it means the most valuable orders can be synced
first when moving the order book to a new matching server. This means
that the entire order book doesn't need to be moved all at once for a
new matching server to have initially useful information.

This design ensures that the order book is synced, and it may be
possible to create a provable chain of trades on top of this where
moving the order book to another server is provable and unlocks
all the fees earned by the previous matching server.

This could be accomplished quite easily if the server builds onto
a hashed chain of orders and signs every new open order as a new link
in the chain. Breaks in the chain are provable, the state easily
stamped, lying would be provable on chain since any user could refute
a link or refute the depth of the chain.

## 3.5. Race conditions

This design avoids race conditions because matching is enforcible
and multiple people aren't racing to fill the same order.
Consideration of DDoS attacks, cryptographically provable order book
syncing with atomic incentives, and proof-of-bandwidth need more work
but I offer these suggestions for open consideration.
