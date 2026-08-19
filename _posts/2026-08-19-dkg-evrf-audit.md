---
layout: post
title: "Serai's Distributed Key Generation Protocol Audited by Least Authority"
---

Serai is a decentralized exchange which uses threshold signatures to sign
transactions on external networks such as Bitcoin, Ethereum, and Monero. In
order to create the keys signed with, Serai designed a
[one-round robust Distributed Key Generation protocol](
  https://serai.exchange/2025/09/26/dkg-evrf-security-proofs.html
). This was a novel work which allows the necessary threshold of validators
to create a new key _while ensuring several critical properties_.

Commonly, Distributed Key Generation protocols require _all_ validators to be
online to perform the protocol, enabling a offline single validator to halt all
progress. Serai's Distributed Key Generation protocol is _robust_, ensuring
_any_ threshold of validators can generate a new key, even if some are offline
or misbehaving. This is critical for Serai's ability to tolerate faults, as may
occur within its decentralized set of validators.

The Distributed Key Generation protocol is premised on a novel form of
Publicly-Verifiable Encryption, inspired by the recent work on
[exponent-Verifiable Random Functions](https://eprint.iacr.org/2024/397).
Publicly-Verifiable Encryption allows each validator to encrypt a share of
the resulting private key to another validator and for anyone to then
efficiently verify the encrypted message is as intended, without revealing
anything else about it. This ensures that even if a validator is offline, and
cannot decrypt their received shares of the private key to proactively confirm
their correctness, that when the validator comes back online, they will be able
to decrypt their shares and participate in signing sessions with the new key.

This work on Publicly-Verifiable Encryption was extremely notable as
historical works made use of bit decomposition, splitting each share of the key
into individual bits, before encrypting just one (or a few) bits at a time.
This caused an encryption of a single share to require several ciphertexts to
represent it. In contrast, our work only requires a single ciphertext, without
introducing any new cryptographic assumptions other than the
Decisional Diffie-Hellman assumption over an elliptic curve. Prior works which
only required a single ciphertext used distinct cryptosystems and required much
stronger assumptions for their security.

In our [prior blog post about this](
  https://serai.exchange/2025/09/26/dkg-evrf-security-proofs.html
), we noted how security proofs for our design had been written. Today, we're
happy to announce our implementation has been
[audited by Least Authority](
  https://leastauthority.com/blog/audit-of-serai-distributed-key-generation/
), experts in cryptography and an ideal candidate for this work. The audit was
facilitated by [MAGIC Grants](https://magicgrants.org), an American 501(c)(3)
which funds research and development of noteworthy projects in the
cryptocurrency and privacy space.

We invite those curious to read the audit itself, but as a brief summary, two
low-severity issues were found and resolved. Additionally, we would like to
note Least Authority complimented our code's quality, a goal we strive for.
Quality does not exist for quality's sake, but instead reduces the risk of
bugs and unintended behavior, while enabling review, promoting the project's
security and integrity. These properties are of the utmost importance to Serai.

Our implementation is positioned as a public good, eligible for use by anyone
who needs an Distributed Key Generation protocol within their application. We
hope others may find value in it and it can benefit the space as a whole, now
that it's had the necessary external review to suggest such usage. It's built in
Rust upon the
[generalized-bulletproofs](
  https://github.com/monero-oxide/monero-oxide/tree/fcmp%2B%2B/crypto/generalized-bulletproofs
) library, as hosted by the
[monero-oxide](https://github.com/monero-oxide/monero-oxide) project and
originally intended for use within Monero's upcoming FCMP++ protocol upgrade.
Accordingly, the generalized-bulletproofs library has itself been
[reviewed and audited](
  https://github.com/monero-oxide/monero-oxide/tree/fcmp%2B%2B/audits/generalized-bulletproofs
), providing a strong base for us to rely on.

Serai will continue to have its code reviewed and tested, leading up to its
next testnet which we hope is soon. If you would like to keep an eye, we
welcome you to follow Serai on
[Twitter](https://twitter.com/SeraiDEX), or join our community via
[Discord](https://discord.gg/mpEUtJR3vz) or
[Matrix](https://matrix.to/#/#serai:matrix.org).
