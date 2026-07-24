---
layout: post
title: "Achieving Consensus"
---

Serai attempts to maintain its security even under asynchrony, a state where an
adversary decides when messages are received and in what order. This means the
adversary may publish a hundred blocks onto the Serai network, with no activity
on any other networks, or may publish thousands of events onto other networks
while the Serai network makes no progress. No matter what occurs, and when it
occurs, it is the responsibility of Serai's validators to agree on a canonical
timeline.

In order to not burden the Serai blockchain, only the results of our consensus
protocols are published on-chain. The process of achieving consensus over the
order of events is handled entirely off-chain. This not only minimizes the Serai
blockchain's storage requirements but is a key reason why the Serai network may
so efficiently handle external networks: each external network has its own
dedicated consensus protocol with only the relevant validators participating.
While that is the premise of Serai's architecture, there's still the question of
how do we _actually_ achieve consensus?

Serai looked for the following properties from a consensus protocol:
- Simple. This part of Serai does not need to achieve impeccable latency or the
  utmost throughput in order to establish a well-functioning, secure
  decentralized exchange.
- Lightweight. The consensus protocol should not be too expensive to run and
  should be feasible to maintain the state of.
- Embeddable. The consensus protocol should be a library within a Serai service,
  not its own service, so that we may easily spawn, simultaneously run, and
  teardown instances.

To this end, we wrote
[our own implementation](https://docs.rs/tendermint-machine) of
[Tendermint](https://arxiv.org/abs/1807.04938) in Rust. Tendermint is a very
straightforward protocol with years of academic review and discussion, and does
not require any prior setup (such as a
[Distributed Key Generation (DKG) protocol](
  https://serai.exchange/2025/09/26/dkg-evrf-security-proofs.html
)). The Tendermint consensus protocol is also of notable fame due to its usage
within the Cosmos ecosystem, for which their implementation used to share a name
with the protocol itself (though it's since rebranded to
[CometBFT](https://github.com/cometbft/cometbft)). We do not use CometBFT
ourselves as we have quite different design goals. We also do not implement a
version of Tendermint compatible with CometBFT, instead focusing on the protocol
itself.

Our implementation focuses on a clear seperation between consensus and the
application, the latter actually defining the blockchain. This is similar to how
CometBFT defines its Application Blockchain Interface (ABCI), allowing the
application to apply consensus to whatever data and rules it wants under a
highly-abstracted model. Our own implementation is also _database-agnostic_,
allowing usage with the database already being used by the application, and
_network-agnostic_. Serai's own usage currently instantiates Tendermint over
RocksDB and [libp2p](https://libp2p.io/)'s
[gossipsub](
  https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/README.md
) protocol. Finally, our implementation is agnostic to the cryptography,
allowing the use of one's preferred signature scheme or even threshold
signatures (if one does choose to perform a Distributed Key Generation
protocol), and the definition of a validator itself, allowing anything from
public keys to a numeric index to usernames. The point is for this
implementation to be able to fit into whatever box your app needs, offering a
simple and direct way to achieve consensus.

The implementation itself operates with bounded memory approximately linear to
the amount of validators, making it highly resistant to Denial-of-Service
attacks. Many APIs are explicitly asynchronous in order to not block the
process, prioritizing the application's efficiency and throughput, yet the
library does not require any specific asynchronous runtime, supporting both
single-threaded and multi-threaded designs. The library is also 100% documented,
if you would like to pick it up and try it out. While it is not currently
covered under [Serai's Bug Bounty Program](https://immunefi.com/bounty/serai),
and is still considered under development, we plan to continue to stablize it
and offer such guarantees in the future. We welcome feedback, especially over
some of the remaining
[open design questions](https://github.com/serai-dex/serai/issues/1249).

In the future,
[Serai is interested in _asynchronous_ consensus](
  https://github.com/serai-dex/serai/issues/333
). Tendermint is only partially-synchronous, making it sound even over an
asynchronous network, but it's only live (and makes progress) during periods of
sufficient synchrony (where enough messages are delivered to enough validators
in a sufficiently timely fashion). In contrast, asynchronous consensus is both
sound _and live_ over asynchronous networks, increasing resiliency, and allowing
validators to feasibly run over less stable/highly latent networks such as Tor.
Unfortunately, asynchronous consensus is a much more complicated problem to
solve, causing such efforts to have been postponed until after Serai performs its
initial release.

All of this is just one part of how Serai aims to offer a secure, efficient
service to swap between the Bitcoin, Ethereum, and Monero networks. There are
many hard problems to such a task, where Serai aims to comprehensively solve
each problem, without ignoring any. If you'd like to keep an eye on us or
participate, please follow us on [Twitter](https://twitter.com/SeraiDEX) or join
our
[Discord](https://discord.gg/mpEUtJR3vz)/[Matrix](https://matrix.to/#/#serai:matrix.org).
