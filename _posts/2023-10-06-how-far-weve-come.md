---
layout: post
title: "How Far We've Come"
---

Serai is a cross-chain decentralized exchange which will be capable of swapping BTC, ETH, DAI, and XMR. Having been under development for roughly two years, it's been an exhausting journey. We've known what we have to do, found solutions, found new problems, and found even more solutions. Despite all the challenges faced, we've managed to overcome them and produce a _highly versatile_ stack ready to face its challenges now _and_ be amenable to all the challenges we can think to throw at it in the future. With it approaching readiness, it's time to start explaining how all the pieces fit together to be the extremely efficient system it is.

## The Blockchain

Serai itself is a blockchain, built on top of Substrate, a blockchain development framework comparable to the Cosmos SDK. The main reason for using a blockchain framework is because Serai is not attempting to revolutionize blockchain networks with advanced consensus mechanisms and new methods of P2P communication. Serai is attempting to be a rock-solid decentralized exchange, and using a blockchain to coordinate token and liquidity pool state is an implementation detail. Substrate is sufficiently performant for our needs, with a notable ecosystem built around it.

## Validators

Validators stake to participate in consensus, either Serai's own or consensus over an external network. Since validators stake per network, at launch we will be looking at having _up to 600 validators_. With this capacity, we can ensure all validators who want to participate can actually do so, never leaving stake unused due to not being selected to actively participate. We also can drastically bring down the node requirements in comparison to other networks as validators _do not_ have to run nodes for every single blockchain. They solely have to run a Serai node and nodes for any external networks they choose to validate over.

## The Coordinator

Each validator who validates over an external network runs a "coordinator". The coordinator is a service which forms a P2P network with all other validators' coordinators, enabling performing signing operations. While we could reuse the Serai blockchain for this, doing so would bloat the blockchain with ephemeral data we don't need nor want to keep around forever.

We do however need to obtain consensus over how signing operations occurred. If any validator was allowed to publish multiple messages within a signing protocol, it'd cause a variety of security issues. In order to ensure everyone sees all messages, and that each validator only published a single message, we place all messages for each signing protocol into the mempool of a disposable blockchain known as a "tributary". Validators produce blocks containing these messages, coming to consensus on them. Only messages with consensus are acted on, ensuring that no validator sent distinct messages to different validators without detection.

At the end of a multisig's lifetime, its blockchain is disposed of, ensuring we don't waste storage space on old signing protocols that have run their course.

## The Processor

Each validator for an external network runs a "processor" for it. Processors index the external network, find relevant transactions, inform Serai of any received coins, and schedule payments out from Serai.

This is arguably the most critical part of Serai. Our entire purpose is to connect with external networks and the processor must do so with complete accuracy. Any mistake when indexing could lead to a double spend against Serai. Any hiccup in how we sort data could cause processors to disagree on the current state, leading to consensus failures.

To ensure security and propriety, we've clearly modeled our data pipelines and have extensively worked to ensure we're free of race conditions. We've even begun exhaustively modeling Serai's design to _prove_ it free of race conditions.

We also have audited the cryptography used by our processor, audited our application of our cryptography to Bitcoin, and are planning audits for our application to Ethereum/Monero, along with an audit for the processor itself.

## Serai

All of these work in concert to become Serai, a platform priding itself on correctness and efficiency, intending to offer the most competent cross-chain decentralized exchange out there.

If you have any questions, or simply want to keep an eye on Serai, please join us on [Discord](https://discord.gg/mpEUtJR3vz)/[Matrix](https://matrix.to/#/#serai:matrix.org)/[X (Twitter)](https://twitter.com/SeraiDEX).
