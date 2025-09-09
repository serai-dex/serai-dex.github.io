---
layout: post
title: "Announcing, monero-oxide!"
---

In April of 2022, Serai began development on monero-serai. Originally
attempting to build a library to facilitate multisignature signing of Monero
transactions, using the Monero C++ codebase and the existing monero-rs
project, monero-serai soon became a completely independent implementation of
the Monero transaction protocol. It had the goal of being usable throughout the
entire Monero community, not just the Serai protocol, offering a modern
representation of transactions, signing, and even an efficient multisignature
protocol.

In February of 2023, the Cuprate project began, striving to build a complete
implementation of the Monero protocol in Rust. Later that same year, they adopted
monero-serai to represent transactions and validate Monero's zero-knowledge
proofs, contributing back implementations of legacy aspects of the Monero
protocol and helping to ensure the library's quality.

Due to monero-serai intending to benefit the community, a request was made
for the Monero community to crowdfund an audit. Raising the funds, Cypher Stack
was contracted to prove the security of our multisignature protocol audit
monero-serai. This produced [FROSTLASS](
  https://github.com/monero-oxide/monero-oxide/tree/main/audits/FROSTLASS
), a formalized and proven-secure threshold signing protocol for CLSAGs
inspired by [FROST](https://eprint.iacr.org/2020/852), and the
[audit of monero-serai itself](
  https://github.com/monero-oxide/monero-oxide/tree/main/audits/Cypher%20Stack%20May%202025
). FROSTLASS offers a major performance improvement over the multisignature
implementation inherently present in Monero, and now formally proven, can be
strongly recommended for implementation by anyone who wants to secure Monero
within a threshold multisig.

Audit in-hand, to ensure monero-serai would remain a neutral library,
monero-serai was transferred to the recently-formed
[monero-oxide organization](https://github.com/monero-oxide). The organization
is led by boog900, maintainer of Cuprate, and kayabaNerve, lead developer of
Serai. Together, they're trusted to steward it for the benefit of the whole
Monero community. Our upcoming goal will be a 1.0 release, representing a
degree of stability over our APIs.

In the future, monero-oxide will also take responsibility for the Rust
libraries underpinning Monero's [Full-Chain Membership Proofs++](
  https://web.getmonero.org/2024/04/27/fcmps.html
) upgrade, prior maintained personally by kayabaNerve. The
generalized-bulletproofs library, implementing the zero-knowledge proof
underlying FCMPs, is expected to be
[merged into the monero-oxide repository in the near future](
  https://github.com/monero-oxide/monero-oxide/pull/39
).

By the generosity of [Power Up Privacy](https://powerupprivacy.com/), we are
also able to announce a [$100,000 bug bounty for monero-oxide](
  https://immunefi.org/bug-bounty/monero-oxide
). Payouts will be in XMR and entirely handled by Power Up Privacy. We offer
this to prioritize our implementation's security and accuracy, welcoming
security researchers to take a look. Our codebase includes Rust code,
zero-knowledge proofs, multi-party protocols, and soon even elliptic curve
implementations, offering a plethora of topics to review.

If you wish to keep an eye on, or discuss, monero-oxide, please join
[Cuprate's Matrix](https://matrix.to/#/#cuprate:monero.social), which currently
hosts discussions for the monero-oxide organization. We look forward to seeing
you there!
