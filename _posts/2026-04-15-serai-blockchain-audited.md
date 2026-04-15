---
layout: post
title: "Audit of Serai's Substrate Blockchain"
---

Serai is a decentralized exchange with integrations for Bitcoin, Ethereum, and
Monero. In order to fulfill this purpose, it is composed of many services,
including a Substrate-based blockchain. Our blockchain serves as a decentralized
network to select validators for the Serai protocol, provide consensus over
external networks, and decide the ordering to execute swaps.

We chose Substrate early on so Serai did not have to build its own blockchain.
While we could have, the point of Serai was to offer a quality user
experience to swap between Bitcoin, Ether, DAI, and Monero, where our usage of a
blockchain was an implementation detail, not the purpose itself.
[Substrate](https://github.com/paritytech/polkadot-sdk/tree/master/substrate),
now considered part of the Polkadot SDK, offered an expansive, flexible
blockchain SDK built by Parity and usable for our purposes, where we would not
have to maintain the database, networking layers ourselves.

Despite being built with Substrate, Serai has no direct relation to Polkadot and
is its own network entirely. As our use of a blockchain is an implementation
detail of Serai, the usage of Substrate is also considered an implementation
detail of the blockchain. To this end, we've worked extensively to define our
own [ABI](https://github.com/serai-dex/serai/tree/next/substrate/abi) and JSON-RPC API,
ensuring we define the protocol fundamentally used for the lifetime of the
project and have control over the entire experience, from developers to users.
One notable feature to this end has been our own transaction format, allowing
users to atomically perform multiple actions within a single transaction, with
guarantees on when the transaction is executed and the effects of it.

In January, we announced this blockchain's
[upcoming audit](
  https://serai.exchange/2026/01/14/serai-blockchain-audit.html
) by [Security Research Labs](https://srlabs.de). SRLabs are experts within the
Polkadot ecosystem, including for Substrate, and have worked for years to ensure
its security. We found such review critical not only due to how important our
blockchain is to our protocol, but also because of the depth of the work we had
done with Substrate as a blockchain framework. We are exceptionally happy to
have benefited from SRLabs participation in Serai's security, them having
contributed many valuable insights and caught many bugs and subtleties, which
will be covered in the remainder of this blog post.

This audit was primarily conducted in the first few months of 2026, with the
findings reported in real time to
[Serai's GitHub issues](https://github.com/serai-dex/serai/issues). Serai's
GitHub issues are not only how the project itself primarily organizes, naturally
indexing the findings, but also served to ensure transparency. Anyone could see
opened issues as they were found and inspect how they were handled.

This review was conducted incrementally, with certain sections of the codebase being submitted after others had already been reviewed. This helped with the review's timeline, but due to scheduling issues on our end, some sections were submitted without our intended degree of testing. This can be attributed as the primary underlying reason for some of these findings, highlighting the importance for Serai to ensure its code is ready and completely through internal processes before being sent for external review.

SRLabs's report notes 21 issues, their framework classifying issues as
"Critical", "High", "Medium", "Low", or "Informational". No issues in the report
had a "Critical" classification, all being "High", "Medium", "Low", or
"Informational". While we can be happy with how we proactively engaged with
SRLabs, working towards a codebase we can trust to be safe and reliable prior to
any deployment, the amount of issues found during external review (and the
severity of some) mean we must work towards higher standards in the
future. We must ensure issues such as these do not continue to occur and that
any remaining issues are thoroughly identified and mitigated. While
security reviews and audits are part of our processes (as incredibly justified
here), we must always be mindful of what may slip through the cracks and
determined to always be vigilant accordingly. All following uses of these
labels for classifying severity will be as assigned within the audit.

We would like to thank [MAGIC Grants](https://magicgrants.org) for helping to
facilitate the audit and Power Up Privacy for their donation which funded it.

### Mitigation Status

All issues were responded to, with accepted mitigations, except four. We would
like to explicitly cover these four issues to be exceptionally clear on their
status.

- A
  [Medium-severity potential denial-of-service](
    https://github.com/serai-dex/serai/issues/744
  ) is considered open, with us planning to resolve it in the future. We have
  already publicly communicated our
  [planned mitigation](
    https://github.com/serai-dex/serai/issues/744#issuecomment-4011521794
  ) (the addition of policy filters to a mempool which nodes may configure in
  response to any environment Serai may eventually find itself in).
- A
  [Low-severity potential denial-of-service](
    https://github.com/serai-dex/serai/issues/772
  ) was responded to as intentional, as it aligns with our protocol's view of
  storage as intended to enable pruning old state in the future.
- One [Informational concern](https://github.com/serai-dex/serai/issues/774) was
  raised regarding the ability for anyone to sweep coins sent to addresses not
  intended to hold coins. We accepted it as-is due to the only impact being how
  coins which would be lost forever may instead be recovered by everyone.
- One [Informational finding](https://github.com/serai-dex/serai/issues/776)
  noted how some areas of the Substrate node (such as the environment it's
  deployed in and some denial-of-service mitigations) was marked for future
  consideration and therefore could not be included within the current review.

### Denial-of-Service Findings

Of all 21 reported findings, we consider 12 of them to either be a denial of
service or performance degradation in some regard. Each's severity was dependent
on how accessible the issue was and the impact of it. We present all of them in
a single section here. The brief descriptions represent Serai's own summary and
is not intended to be presented as a quote from SRLabs or their report.

- S3-727 (High): Certain invalid messages could cause the program to panic
  (abort). This was reported as High severity due to presumably being reachable
  over the network and has been fixed with proper error propagation.
- S3-729 (High): Errors from the key aggregation performed, if reached, would
  cause any operations requiring that aggregation to stall due to the assumption
  all aggregation errors were unreachable. Additionally, the method in which it
  would fail could degrade the node's performance and cause disruptions. Each
  error is now individually propagated before being justified as unreachable,
  with one case requiring additional checks to ensure this.
- S3-753 (High): The initial call for a validator set to set its keys would
  error due to invalid session indexing, causing the validator set to be unable
  to set its keys, preventing its operation.
- S3-759 (High): An off-by-one error in the scheduling for the addition of coins
  used to form Protocol-Owned Liquidity would cause the chain to halt.
- S3-760 (High): Differences in when a truncating division was applied (to
  liquidity when queued vs. to all queued liquidity) could cause the chain to
  halt.
- S2-733 (Medium): The configuration for transactions, ordered by nonces, in the
  mempool was inverted, preventing any transaction which followed a prior
  transaction from co-existing with its parent in the mempool.
- S2-744 (Medium): This was the prior-noted denial of service which remains open
  for now, with a mitigation planned (mempool-based policies), but not yet
  implemented.
- S2-746 (Medium): Swaps for a certain amount out could trigger a runtime panic,
  causing the swap action to fail in a way enabling degrading the node's
  performance.
- S1-745 (Low): Certain deallocations could trigger a runtime panic, causing the
  deallocation to fail in a way degrading the node's performance. The cited code
  was an artifact of the development process and was not present in the version
  officially sent for review.
- S1-747 (Low): An extremely unbalanced pool, such as one with the maximum
  possible amount of SRI yet the smallest possible amount of an external coin,
  could cause a valuation function to panic. Depending on where that function is
  called, this may or may not have been usable to halt the chain, though the
  conditions required to trigger it are likely unreachable in any real-world
  setting.
- S1-772 (Low): Historic state in a certain section of the protocol can not
  actively be removed, despite how it can be actively removed while still
  relevant. This is the aforementioned finding we considered intentional. The
  eventual goal is to allow nodes to clean up historic state themselves, which
  requires historic state truly be historic, with no further accesses nor usage
  on-chain. By actively allowing users on-chain to remove historic storage,
  those values technically cannot be considered historic, as they may still be
  interacted with on-chain, preventing them from being cleaned up as historic by
  any future processes.
- S0-757 (Info): When the network launches, anyone may add external coins to
  become 'genesis liquidity'. As Serai's liquidity pools are symmetric, all
  liquidity is composed of both an external coin and SRI. In order to actually
  add the genesis liquidity to the pools, the protocol will airdrop the
  corresponding SRI portion to those who added external coins. In order to know
  how much SRI to use for sriBTC liquidity, and how much to use for sriETH,
  sriDAI, sriXMR liquidity, the values of one to another are oraclized onto the
  network by the genesis validators who start the network.
  
  This finding noted that this oraclization process used the latest keys
  declared by the genesis validators, not the keys declared upon genesis. This
  allowed a genesis validator to sign a signature before updating their key,
  invalidating the signature (due to being signed by an old key), and forcing
  the genesis validators to have to sign a new signature. This was mitigated by
  ensuring we use the keys declared upon genesis.

Serai, as a project, generally considers undocumented, reachable panics in any
public API as a security issue. This is reflected in our long-standing
[Bug Bounty Program with Immunefi](https://immunefi.com/bug-bounty/serai/scope),
where even when the caller crafts arguments in a way intended to cause panics,
we still consider the code panicking an issue and offer rewards. We are
generally proud of covering our work under such a rigorous criteria, and to that
end, acknowledge and accept all of the above issues.

Practically, S3-727 and S3-729 (each assigned "High" severity) are the most
notable in our opinion. S3-753, S3-759, S3-760, are notable as they should have
been caught with testing and highlight a need for us to continue to write and
expand our test suite, for all aspects of our codebase.

### Non-Denial-of-Service Findings

With 12 of 21 findings considered as denial of services, or performance
degradations of some form, nine can be considered as logical bugs affecting or
questioning the soundness of the protocol. This section will cover each of these
in more detail due to how each bug could potentially have had much greater
impact. Thankfully, of these nine findings, only two were assigned "High"
severity, with three assigned "Medium", one assigned "Low", and the remaining
three assigned an "Informational" severity.

- S3-741 (High): This was interesting as it was a flaw in our
  [Economics design](
    https://github.com/serai-dex/serai/blob/next/spec/Economics.md
  ). This review process led to multiple improvements of the mechanisms
  posited, the resulting document (the one linked) intended to be near-final if
  not final. Barring any documentation errors, or sections deserving clarification,
  the only sections actively being considered for any further changes are the exact
  fees proposed for deployment.
  
  Before the network achieves Economic Security, the phase where the validators'
  stake's value exceeds the value of the coins tracked on the Serai blockchain
  itself (with a certain additional buffer), users are allowed to swap external
  coins to staked SRI in order to form validators. While SRI may always be
  swapped to via the liquidity pool, and anyone with sufficient SRI may allocate
  stake to become a validator, swapping to staked SRI is a specific
  functionality which only exists prior to achieving Economic Security. While
  swapping through a liquidity pool would cause the quote to change in response
  to the swap itself, swapping to staked SRI _mints_ SRI directly as stake
  allocated to a validator at a specific quote (without impacting the quote).
  This makes becoming a validator much more stable and accessible, promoting
  becoming a validator so that the network may become Economically Secure.

  The external coins which are used to swapped to staked SRI are then used to
  form Protocol-Owned Liquidity. This process, both as originally and currently
  specified, involves acquiring liquidity from the existing liquidity pool in
  exchange for the external coins.

  This finding, when paraphrased, covered how users who supply liquidity _and_
  swap to staked SRI could effectively do so at a discount. While the external
  coins would be used to form Protocol-Owned Liquidity, part of the liquidity
  acquisition would be from the user's own liquidity position. This would cause
  the user to swap to staked SRI as expected, yet effectively receive back a
  portion of the external coins (proportional to how much of the pool's
  liquidity they had). We mitigated this issue by ensuring that while users are
  always able to remove their liquidity, they only receive any upside according
  to a trickle feed which only begins after Economic Security is achieved. This
  is to ensure a steady and predictable flow for value which all users may
  observe and decide how to react accordingly to, without any sudden changes.

- S3-758 (High): The code for validating misbehavior in the GRANDPA consensus
  code had an inversion such that any alleged misbehavior (even if invalid)
  would be accepted as evidence of misbehavior. This was a complete oversight
  present due to when the code for misbehavior was added, and due to how it
  unfortunately lacked tests when submitted.

- S2-761 (Medium): When forming Protocol-Owned Liquidity, the coins used to
  acquire liquidity are dripped into the pools. If the amount of these coins is
  less than the amount of blocks to drip over, the rate per block may be zero,
  where any remainder is expected to be added with the final block of the drip
  feed. If the drip rate was zero however, the code incorrectly interpreted it
  as there being no liquidity to drip, potentially not adding some coins into
  the pools as expected. This was mitigated by correcting how we check for if
  there's outstanding liquidity to drip into the pools, as suggested.

- S2-771 (Medium): Serai does not have a premine nor any fund for development
  enshrined into the protocol. Serai also lacks the on-chain upgrade
  functionality frequently seen with projects built with Substrate, believing
  the only code which should run is that which the whoever runs the node
  downloads and explicitly consents to. To this end, Serai lacks anything we
  would call or consider approximate to a Decentralized Autonomous Organization
  (DAO).
  
  Serai does have 'signals' however, where users may express favor for a signal.
  A signal may be either to halt an external network, if an issue is perceived,
  or to retire the network itself. These signals only cause a corresponding
  effect when a sufficient amount of the protocol's validators actively express
  favor for the signal.
  
  This issue noted that while expressed favor for a signal intended to have a
  limited lifespan, specifically until the end of the following session of Serai
  validators, the favor's lifetime could potentially last until the end of any
  validator set which overlapped with the following session of Serai validators.
  In practice, this meant a lifetime intended to be for just one to two weeks
  could actually last two to three weeks.
  
  While potentially not the most impactful finding, we greatly appreciate the
  focus applied to find and report it. The challenge with writing the code was
  organizing around the schedules of multiple different sets of validators, all
  who may update independently and seemingly arbitrarily. While this oversight
  did occur, we are thankful SRLabs put in the effort necessary to identify even
  more delicate oversights such as this. As we do not consider Serai a product but a project, aiming to capture an ideal of correctness and security missing throughout the space, SRLabs also considering Serai as a protocol and thoroughly holding it the same ideal was above and beyond any expectations we could have had.

- S2-763 (Medium): When an external network is halted, only the external
  network's integration's ability to publish new events was halted. This
  prevented any new coins from being minted on-chain and prevented the associated
  validators from being able to rotate to the next set. This finding noted how
  other aspects, such as the ability to swap to coins associated with the
  'halted' external network, remained possible.
  
  Originally, this was not considered an issue, as the coins on the Serai
  network were still coins on the Serai network, with all on-chain abilities
  intact. While further transfers, swaps, burns could complicate how these coins
  are tracked, any observations after the fact would _have_ to develop
  comprehensive tracking and accounting solutions regardless to ensure their
  correctness. Accordingly, the impact of this could be considered non-existent.
  
  Two specific ideas were raised in the following discussions however which
  justified this finding. The first was that if an external network's
  integration is halted due to a bug, the fact the coins on the Serai network
  could still be burnt meant the external integration could still be instructed
  to take action (despite being 'halted'). It was only if there was a bug or
  issue with reporting events onto Serai that the halt would be effective. While
  our integrations should be programmed to observe a halt and stop on their own,
  meaning applying such a methodology on-chain would be unnecessary, it was
  still agreed worthwhile to be so robust with our methodology. The other
  concern was how swapping to coins is intended to specify a minimum amount to
  receive, yet if the external network has been halted, one can question if
  those coins are the same and should be considered as having the same amount
  for the purposes of enforcing a minimum value out when swapping. To ensure
  both of these were handled optimally, halts were expanded to not halt just the
  publication of events, but to also halt burns of and swaps to related coins.

- S2-742 (Low): One of our build scripts output the first binary found with the
  expected name, instead of the binary which was a result of that build process.
  If the build directory was dirty, or in exceptionally strange cases such as a
  build process which outputs interim artifacts with the same name as the final
  artifact, this could cause the wrong binary to be selected. This has been
  fixed by adding a JSON parser to properly parse the Rust compiler's output in
  order to identify the location of the result.

- S0-731 (Info): This finding noted how coins are associated with a network, yet
  any coin could be specified in any context, even when that context was
  associated with a distinct network. This finding was questioned when received
  as in general, contexts are always derived from the immediately present data.
  For example, when given a coin, one may call a method to obtain its network,
  or vice-versa. As the network of a coin is baked into the code itself, and
  cannot be manipulated, the only way for such a potential mismatch to occur
  would be in a location where _both_ a network and a coin was given.
  
  Because being provided with two pieces of data which can conflict is a risk
  that they may conflict, the Serai codebase is written to avoid such scenarios,
  preventing this issue from ever appearing. In response to this finding, we
  checked the entirety of the in-scope code for any such cases, and only
  identified one place this _could_ occur. This was in our handling of batches,
  where batches represent a series of events from an external network. The
  batches are specified by network, as the batches may not contain any events
  regarding coins. If the batch does have events regarding coins, each event
  specifies the associated coin, as required due to how one network may
  correspond to multiple coins (such as Ethereum, with Ether and DAI). Due to
  specifying a network as context, and then specific coins, this could be a
  place where such a conflict could occur.
  
  The code to verify batches did check that for all coins in all events, the
  network of the coins corresponded to the network of the batch. This was
  considered and was never a reachable issue. In response to this issue however,
  we did extend the code which created and decoded batches with checks for
  consistency. This means the batch, the object which creates the opportunity
  for this mismatch, is responsible for ensuring the data does align, not
  any/all consumers.
  
- S0-774 (Info): This is the aforementioned issue where coins which would be
  otherwise inaccessible may be swept by anyone, which we've accepted as-is. For
  the reasoning of this, please read
  [the issue itself](https://github.com/serai-dex/serai/issues/774).

- S0-776 (Info): This is the aforementioned issue which noted some aspects of
  the node, such as mitigations for denial of service, were marked for future
  consideration and therefore not possible to include in the current scope. It is
  left for us to continue to work on and mitigate in the future.

### Recommendations

The report included a series of recommendations for Serai, as a project, to
adopt. One notable suggestion was for
"constrained on-chain governance and upgrade mechanism to enable rapid incident response".
As a project, we consider the defined halting mechanism (which is accessible
on-chain to the active validators) as the project's ability to respond to any
misbehavior. The gap is if the blockchain protocol itself is faulty, as the
halting mechanism may only be applied to external networks, meaning to halt the
blockchain protocol would require a sufficient amount of validators to pull
their plugs. Additionally, even if Serai may halt in response to an issue, a fix
cannot be automatically deployed.

Our response to S2-771 briefly explained the signals functionality, including
the ideology behind it. That commitment to whoever runs a node choosing and
explicitly consenting to the software they run prevents us from adopting an
on-chain upgrade mechanism. Despite this, we will continue to consider how the
project is organized, and if it may be made more efficient/effective without
compromising on user sovereignty nor decentralization.

Another suggestion was to minimize our divergences from the Substrate, as
developed and published as part of the Polkadot SDK, to reduce our maintenance
overhead and likelihood of introducing security issues. The maintenance burden
can definitely be discussed. Historically, Serai maintained an outright fork of
Substrate, yet the difficulty of reconciling it with new releases caused the
process of updating to take multiple days in what was not generally feasible.
Today, we maintain a
[patch set and patcher](https://github.com/serai-dex/patch-polkadot-sdk), which
we've found reasonable to maintain and able to update to new releases within
just an hour or so. It was included in this audit's scope.

As for why we maintain a derivative, the biggest reason is for scale. As an
immediate metric, our Substrate is measured as having 141,980 lines of Rust code
compared to the upstream's 663,615 lines of Rust code. If we used the upstream,
we may actually only compile and run the same amount of code, but with our
derivative, we have clarity on exactly what that code is. This is intended to
benefit our own work to identify the exact behavior and definition of our own
protocol, while also providing a clearer scope to potentially benefit and enable
review in the future _without_ having approximately five times as much work due
to unnecessarily including the parts of Substrate we don't use.

We also apply patches to reduce how many dependencies we have (as part of our
ongoing efforts regarding supply-chain security), have three patches fixing bugs
in Substrate, have four patches adding features to Substrate, and only three
patches which we consider opinionated (patches not generally desirable yet
desirable for Serai as a project specifically). Note that some modifications are
not represented via patches, solely via instructions to the patcher, and would
not be counted by these metrics.

While we can present all of the above justifications for having our own
derivative of Substrate, the recommendation itself remains sound. With every
modification we make, we may fix a problem or improve something, yet we also may
introduce a problem. While we've drastically clarified our scope, we've also
taken responsibility and liability for our scope by defining this methodology.
We must always keep that in mind and be able to justify our future changes.

Other recommendations included adding fuzzing, expanding tests, and writing
documentation, which we agree on and is a focus of ours. One note was on the
organization of a specific code path, the code path itself already having an
open issue tracking it, and a suggestion to adopt pair programming as part of
our internal practices. We agree on the benefits of such a solution, and while
that isn't currently in place, will consider how we may adopt aspects of it in
order to strengthen our processes.

### Moving Forward

We've indexed
[this audit](
  https://github.com/serai-dex/serai/tree/next/audits/substrate/Security%20Research%20Labs%20April%202026
) into our [audits folder](https://github.com/serai-dex/serai/tree/next/audits).
This index accurately presents the exact scope, with references to the relevant
GitHub issues and associated context. We encourage everyone to feel welcome to
review the full report and all the documentation. Our audits folder also
contains our other audits, covering various parts of our cryptography, Bitcoin
integration, Ethereum integration, and more. For our Monero integration, the
libraries which were monero-serai have become monero-oxide, which has been
audited and maintains its own
[audits folder](https://github.com/monero-oxide/monero-oxide/tree/main/audits).

At this time, only one unaudited aspect of the Serai protocol stands out to us:
our implementation of our
[DKG protocol](
  https://serai.exchange/2025/09/26/dkg-evrf-security-proofs.html
). We are still actively considering follow-up reviews for our blockchain, and
whether any other aspects of the project would greatly benefit from external
review prior to deployment (instead of solely our internal review and testing),
but are excited for how close we are to the end of our checklist.

Moving forward, with our Substrate blockchain protocol considered complete (even
though the node has
[some](
  https://github.com/serai-dex/serai/issues/717
)
[features](
  https://github.com/serai-dex/serai/issues/625
)
[remaining](
  https://github.com/serai-dex/serai/issues/744#issuecomment-4011521794
)), we will be prioritizing working towards a testnet. This will be by
continuing with integration tests for all of our services, confirming they work
together as expected in all cases, before an internal test network, before
finally once again preparing public networks. To this end, we recently had tests
for a notable part of our "coordinator" service contributed and merged, with the
other parts' tests actively being developed (and in some cases, already
submitted).

If you'd like to keep up with Serai as its development continues, we welcome you
to join our [Matrix](https://matrix.to/#/#serai:matrix.org) or
[Discord](https://discord.gg/mpEUtJR3vz), the two being bridged to each other to
allow your choice in how you connect. We also maintain a
[Twitter feed](https://twitter.com/SeraiDEX) and our blog supports
[RSS](https://serai.exchange/feed) for those who prefer.
