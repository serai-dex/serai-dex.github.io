---
layout: post
title: "Is Serai Monero Focused?"
---

Serai is a cross-chain decentralized exchange which will be capable of swapping BTC, ETH, DAI, and XMR. Most of the project's attention has been derived from our integration of Monero, and the extensive amount of work we've performed in order to be able to do so. This has made some people believe Serai is Monero-focused. So is it?

## How much work was done to integrate Monero?

[modular-frost](https://github.com/serai-dex/serai/tree/develop/crypto/frost) is our audited implementation of FROST, a threshold signature scheme to produce Schnorr signatures. We applied it to Monero's CLSAG signature scheme in our very own [Monero library (monero-serai)](https://github.com/serai-dex/serai/tree/develop/coins/monero).

Initially, our Monero library utilized monero-rs, a Rust library implementing various structures for working with Monero transactions. monero-rs never implemented signing, leaving monero-serai to do so. We also relied on the existing Monero cryptography in C++ to perform the Bulletproofs needed inside Monero transactions.

Over time, we replaced more and more of monero-rs as it wasn't as ergonomic as we desired. Eventually, monero-serai completely removed monero-rs, instead fully implementing the Monero transaction structures ourselves. We did still use Monero's C++ for some cryptography, until that was also removed for performance benefits.

This left us with a complete implementation of Monero transactions, signing, and scanning in Rust.

We then extended our library with functionality beneficial to Serai, notably:

1) [Featured Addresses](https://gist.github.com/kayabaNerve/01c50bbc35441e0bbdcee63a9d823789): A specification for Monero addresses offering more functionality than the already defined address schemes.

2) [Eventualities](https://docs.rs/monero-serai/0.1.4-alpha/monero_serai/wallet/struct.Eventuality.html): A way to check if an on-chain transaction matches a transaction we intended to sign.

## With all the work done, how could you not be Monero-focused?

We are not Monero-focused. While yes, we did a lot of work to integrate Monero, all of that work was done in a self-contained Monero library. The actual integration with Serai _completely conforms to the expected API_. At no higher level did we add Monero-specific code.

We are a project that respects Monero for being a quality privacy coin, yet we are our own project which is attempting to be the best solution for cross-chain swaps. Our integration of Monero, with all the time it took, may be a love letter to Monero, but it is not a pledge of allegiance.

Serai has also never taken funding from the Monero community.

## There isn't any Monero-specific code?

Correct. There is no Monero-specific code outside of its integration file &#40;and the self-contained monero-serai library, of course&#41;. It satisfies the exact same API we use for Bitcoin, despite being a privacy coin with its own set of complexities. This is possible thanks to:

1) Strong theoretic modeling of requirements to integrate an external network

2) Reduction of those requirements to minimal forms, offering more flexibility with integrations

3) Our own software stack being incredibly robust and versatile

## Why didn't Serai launch earlier with just Bitcoin and Ethereum then?

All of this work on Monero took several months. If we aren't Monero-focused, how can we justify delaying all of Serai for so long in order to launch with it?

Integrating Monero prior to launch has actually been amazing for us. Only due to facing Monero's complexities did we properly define, and revisit, assumptions we place on external networks in order to successfully integrate with them. By ensuring we can support Monero prior to launch, we've ensured and rigorously demonstrated our software stack can face whatever challenge we may line up in the future.

There also wouldn't be notable appeal to Serai if we solely launched with Bitcoin and Ethereum. There's multiple protocols which already offer Bitcoin-Ethereum swaps. By also including Monero, we provide a unique offering and demonstrate our technical competency. We aren't here to be the 20th bridge between Ethereum and all the forks of Ethereum. We are here to connect networks, regardless of their specific challenges, and provide a quality cross-chain decentralized exchange.

We could have launched with a distinct network, one largely unserviced by DEXs, in order to establish our unique offering. We chose Monero, despite the amount of time and effort it would be, due to its importance. In a modern day society which is increasingly digital and increasingly less private, Monero fights to re-establish privacy for your payments. We believe in that mission and want to enable everyone to have access to personal privacy.

## So what does all this mean?

Largely nothing. If you're interested in Serai for its integration of Monero, Serai will still do its best to integrate Monero. Serai will also do its best to integrate Bitcoin and Ethereum, the other networks its launching with, as part of its responsibility to be the best DEX it can be. This post isn't intended to disregard Monero, yet instead to highlight how Serai is a freestanding project with its own goals and responsibilities. With that comes responsibilities to Monero, yet none unique to Monero. Instead, they're the same responsibilities given to all integrated networks.
