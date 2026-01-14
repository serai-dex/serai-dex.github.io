---
layout: post
title: "Serai DEX's Blockchain's Audit Kicks Off"
---

Serai is a cross-chain decentralized exchange with many different components to
it. In the past, we've:

- Implemented the [FROST](https://eprint.iacr.org/2020/852) signing protocol,
  having our implementation [audited by Cypher Stack in 2023](
    https://github.com/serai-dex/serai/tree/cc5d38f1ce03cbb78e0c595d0521ffd3b3c19c2f/audits/crypto/Cypher%20Stack%20March%202023
  ).
- Designed a [one-round robust Distributed Key Generation protocol](
    https://serai.exchange/2025/09/26/dkg-evrf-security-proofs.html
  ) with security proofs written by [HashCloak](https://hashcloak.com/).
- Started what has been spun off into
  [monero-oxide](https://serai.exchange/2025/09/09/monero-serai-oxide.html), a
  Rust implementation of the Monero transaction protocol and wallet
  functionality, audited by [Cypher Stack](https://cypherstack.com) in 2025.
  This included a formal description, and proof of security, for our Monero
  threshold signing protocol, [FROSTLASS](
    https://github.com/monero-oxide/monero-oxide/tree/main/audits/FROSTLASS
  ). FROSTLASS applied the techniques of FROST to
  [Monero's CLSAG ring signature](https://eprint.iacr.org/2019/654), achieving
  a secure, efficient, and scalable solution for decentralized signing of
  Monero transactions. These same premises underpin the threshold signature
  designs for [Monero's upcoming FCMP++ upgrade](
    https://www.getmonero.org/2024/04/27/fcmps.html
  ).
- Had our contracts for our Ethereum integration, and immediately proximate
  Rust libraries,
  [audited by Trail of Bits](
    https://serai.exchange/2025/06/06/ethereum-contracts-audited-by-tob.html
  ).
- Had our [bitcoin-serai library audited by Cypher Stack](
    https://github.com/serai-dex/serai/blob/7d2d739042784466e855e6e3e75cbf0e898b80ff/audits/Cypher%20Stack%20networks%20bitcoin%20August%202023
  ).

While these topics are about how Serai interacts with external networks, be
they Bitcoin, Ethereum, or Monero, there is also the Serai network itself (or
even networks, depending on where one draws the lines). Fundamentally, Serai
uses a blockchain to select validators and order swaps, one built upon the
Substrate framework. The Substrate framework was chosen for being written in
Rust, being incredibly flexible, intending to allow developers to build their
own blockchains, as well as performant. While this meant we did not have to
write a blockchain node implementation from scratch for Serai's blockchain,
which would've been unnecessary and out-of-scope to the Serai project, we did
still have to define and implement the protocol for our specific needs and
functionality.

As of Monday, the audit of our Substrate-based blockchain kicked off, with
[SRLabs](https://srlabs.de/) as our auditor. SRLabs has a long history within
the Substrate ecosystem, giving them unmatched experience. They've previously
developed [tooling to secure the Substrate ecosystem](
  https://github.com/srlabs/substrate-runtime-fuzzer
), with extensive usage over several projects, and maintain a security
consultancy relationship with [Parity](https://parity.io), the company which
primarily developed the Substrate framework.

The audit is being executed in a staged approach, with the first set of
libraries already submitted, and the complete implementation expected to be
delivered in mid-February. This pipelined approach allows us to minimize the
delay on having our works externally reviewed, a crucial requirement for a
secure deployment. We look forward to this engagement with SRLabs and hope to
maintain a relationship moving forward into the future as well.

The audit was facilitated by [MAGIC Grants](https://magicgrants.org), an
American 501(c)(3) which funds research and development of noteworthy projects
in the cryptocurrency and privacy space. They provide scholarships to students
interested in the field, and also host the
[MAGIC Monero Fund](https://magicgrants.org/funds/monero) as a tax-deductible
way to donate to Monero development (along with the
[MAGIC Firo Fund](https://magicgrants.org/funds/firo) and
[MAGIC Privacy Guides Fund](https://magicgrants.org/funds/privacy_guides)).

The funds for the audit were generously donated by
[Power Up Privacy](https://powerupprivacy.com), a charitable entity that
donates to projects which benefit privacy. They've previously donated to Serai
to enable the security proofs for its bespoke Distributed Key Generation
protocol and the audit of its Ethereum contracts.

We are incredibly thankful to both organizations for enabling us to do our best
to ensure the security of the Serai project, which is moving into a final       
state. With our Substrate-based blockchain being audited, and the ongoing
review and testing of our other services, we plan to be able to host testnets
soon after this audit resolves. We look forward to finally demonstrating the
experience Serai enables to swap between various cryptocurrencies, including
Monero, in a decentralized way.

If you would like to keep up to date with Serai as it achieves these
milestones, we welcome you to follow us on
[Twitter](https://twitter.com/SeraiDEX), or join our community via
[Discord](https://discord.gg/mpEUtJR3vz) or
[Matrix](https://matrix.to/#/#serai:matrix.org).
