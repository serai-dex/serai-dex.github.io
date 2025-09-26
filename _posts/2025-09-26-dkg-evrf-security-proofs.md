---
layout: post
title: "Security Proofs for our One-Round, Robust Threshold DKG"
---

Serai is a decentralized exchange premised on threshold multisignatures,
primarily the [FROST](https://eprint.iacr.org/2020/852) signing protocol. To
achieve this, our first step is to actually _create_ the signing key. We do
this with a distributed key generation protocol, or DKG.

The FROST paper posits a DKG, known as PedPoP, which Serai originally planned
to use. PedPoP does not allow identifying a participant who misbehaves however,
so Serai implemented a 'complaint round' into our protocol. After PedPoP
completes on paper, each participant sends a message to say either "I'm good!"
or "Eve sent me an invalid key!", allowing us to detect invalid participants.
Our implementation was initially written independently yet was effective the
protocol from the [ICE FROST paper](https://eprint.iacr.org/2021/1658).

Unfortunately, our DKG protocol still faced an issue: everyone must be online
and remain online for three 'rounds' (every participant sending one set of
messages and receiving the messages from the other participants). With Serai,
the point is to be robust and remain operational even if up to a third of
validators go offline.

Our initial solution was to detect if a validator was offline, and if so,
remove them from the key generation protocol. Unfortunately, this means they
would be _permanently_ excluded from the key and considered as 'offline' for
its entire lifetime, even if they came back online later. While this was
functional and acceptable, it clearly wasn't optimal.

So, in July of 2024, we posited a new protocol for distributed key generation
protocols building on the [eVRF paper's](https://eprint.iacr.org/2024/397).
Using its methodology, we were able to posit an efficient mechanism for sending
_encrypted shares of the signing key_ while allowing _anyone_ to verify their
correctness (an efficient mechanism for Verifiable Encryption).

This allows us to remove the 'complaint round' from our DKG protocol and
ensures that all participants receive valid shares of the signing key,
guaranteeing their ability to participate in signing if online and willing.
Combined with the eVRF paper's own proposal for a one-round DKG (but still
requiring a way to identify misbehaving participants), we were able to achieve
a one-round DKG!

While there are prior works on combining DKGs with Verifiable Encryption for
these results, they generally require non-elliptic-curve cryptography or
repetition (instead of encrypting a 256-bit key share in one message,
encrypting each bit of a key share in 256 messages, each requiring their own
proof of validity). Our methodology is notably for only requiring
elliptic-curve cryptography (the hardness of the Decisional Diffie-Hellman
problem) and supporting encrypting entire key shares in a single message.

Today, we've published the
[security proofs](
  https://github.com/serai-dex/serai/tree/next/audits/crypto/dkg/evrf
) for our protocol, which were contracted from
[HashCloak](https://hashcloak.com/) by
[MAGIC Grants](https://magicgrants.org/), as possible thanks to a donation by
[Power Up Privacy](https://powerupprivacy.com/)!

We also have
[implemented our protocol in Rust](
  https://github.com/serai-dex/serai/tree/next/crypto/dkg/evrf
), published under the MIT license. While our implementation has yet to be
audited, it builds upon the `generalized-bulletproofs`,
`generalized-bulletproofs-ec-gadgets` libraries for Bulletproofs and efficient
elliptic curve cryptography within them. These were audited by the Monero
project for usage within their upcoming FCMP++ hard fork. This provides a
strong base for them, helping ensure its foundation.

This marks one more step forward to building a secure, trustworthy, and robust
decentralized exchange for Bitcoin, Ethereum, and Monero. If you wish to learn
more, please visit our [website](https://serai.exchange) or join us on
[Discord](https://discord.gg/mpEUtJR3vz) on
[Matrix](https://matrix.to/#/#serai:matrix.org). We look forward to seeing you
there!
