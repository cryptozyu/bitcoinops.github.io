---
title: 'Bitcoin Optech Newsletter #229'
permalink: /en/newsletters/2022/12/07/
name: 2022-12-07-newsletter
slug: 2022-12-07-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter describes an implementation of ephemeral anchors
and includes our regular sections with the summary of a Bitcoin Core PR
Review Club meeting, announcements of new releases and release
candidates, and descriptions of notable changes to popular Bitcoin
infrastructure projects.

## News

- **Ephemeral anchors implementation:** Greg Sanders [posted][sanders
  ephemeral] to the Bitcoin-Dev mailing list that he's implemented his
  idea for ephemeral anchors (see [Newsletter #223][news223 anchors]).
  [Anchor outputs][topic anchor outputs] are an existing technique made
  available by Bitcoin Core [CPFP carve outs][topic cpfp carve out] and
  used in the LN protocol to ensure that both parties involved in a
  contract can [CPFP][topic cpfp] fee bump a transaction related to that
  contract.  Anchor outputs have several downsides---some of them
  fundamental (see [Newsletter #224][news224 anchors])---but others
  that can be addressed.

  Ephemeral anchors build on the [v3 transaction relay proposal][topic
  v3 transaction relay] to allow v3 transactions to include a
  zero-value output paying a script that is essentially `OP_TRUE`,
  which permits that transaction to be CPFP fee bumped by anyone on the
  network with a spendable UTXO.  The fee-bumping child transaction
  can itself be RBF fee bumped by anyone else with a spendable UTXO.
  In combination with other parts of the v3 transaction relay
  proposal, it is hoped that this will eliminate all policy-based
  concerns about [transaction pinning attacks][topic transaction
  pinning] against time-sensitive contract protocol transactions.

  Additionally, because anyone can fee bump a transaction containing
  an ephemeral output, it can be used for contract protocols involving
  more than two participants.  The existing Bitcoin Core carve out
  rule only reliably works for two participants and [previous
  attempts][bitcoin core #18725] to increase it required an
  arbitrary upper limit on participants.

  Sanders's [implementation][bitcoin core #26403] of ephemeral anchors
  makes it possible to begin testing the idea along with the other v3
  transaction relay behaviors previously implemented by that
  proposal's author. {% assign timestamp="2:03" %}

## Bitcoin Core PR Review Club

*In this monthly section, we summarize a recent [Bitcoin Core PR Review Club][]
meeting, highlighting some of the important questions and answers.  Click on a
question below to see a summary of the answer from the meeting.*

[Bump unconfirmed ancestor transactions to target feerate][review club 26152]
is a PR by Xekyo (Murch) and glozow that improves the accuracy of the wallet's fee
calculation in the case that unconfirmed UTXOs are selected as inputs.
Without the PR, the fee is set too low if the
feerates of some unconfirmed transactions being used as inputs are lower
than that of the transaction being constructed. The PR fixes
this by adding enough fee to "expedite" such low-fee source
transactions to the same feerate as targeted by the new transaction.

Note that even without this PR, the process of coin selection will
try to avoid spending from low-feerate unconfirmed transactions.
This PR will be beneficial in cases where this can't be avoided.

Adjusting the fee to account for these ancestor transactions turns out
to be similar to choosing which transactions to include
in a block, so this PR adds a class called `MiniMiner`.

This PR review [spanned][review club 26152] two [weeks][review club
26152-2]. {% assign timestamp="25:23" %}

{% include functions/details-list.md
  q0="What problem does this PR address?"
  a0="The wallet fee estimation doesn't take into account that it may also
      need to pay for all unconfirmed ancestors with a lower feerate than the target."
  a0link="https://bitcoincore.reviews/26152#l-30"

  q1="What does a transaction’s \"cluster\" consist of?"
  a1="The set of transactions consisting of itself and all
      \"connected\" transactions. This includes all of its ancestors
      and descendants, but also siblings and cousins, i.e. children of
      parents who may not be ancestors nor descendants of the
      given transaction."
  a1link="https://bitcoincore.reviews/26152#l-72"

  q2="This PR introduces `MiniMiner` which duplicates some of the
      actual miner's algorithms; would it have been better to
      unify these two implementations through refactoring?"
  a2="We only need to operate on a cluster and not the entire mempool,
      and don't need to apply any of the checks that `BlockAssembler`
      does. It was also suggested to do this calculation without
      holding the mempool lock. We'd also need to change the block
      assembler to be tracking bump fees rather than building the
      block template; the amount of refactoring necessary was
      equivalent to rewriting."
  a2link="https://bitcoincore.reviews/26152#l-94"

  q3="Why does the `MiniMiner` require an entire cluster? Why can’t it
      just use the union of each transaction’s ancestor sets?"
  a3="Some of the ancestors may already have been paid for by some of
      their other descendants; they don't need to be bumped further.
      So we need to include these other descendants in our calculations."
  a3link="https://bitcoincore.reviews/26152#l-129"

  q4="If transaction X has a higher _ancestor feerate_ than independent
      transaction Y, is it possible for a miner to prioritize Y
      over X (that is, mine Y before X)?"
  a4="Yes. If some of Y's low-feerate ancestors have _other_ decendants that
      are high feerate, Y doesn't need to \"pay\" for those ancestors.
      Y's ancestor set is updated to exclude those transactions,
      which has the effect of increasing Y's ancestor feerate."
  a4link="https://bitcoincore.reviews/26152#l-169"

  q5="Can `CalculateBumpFees()` overestimate, underestimate, both, or
      neither? By how much?"
  a5="It will overestimate if two outputs with overlapping ancestry are
      chosen, since each bumps its ancestors independently (without taking the shared
      ancestry into account). The
      participants concluded that it's not possible for bump fees to be
      underestimated."
  a5link="https://bitcoincore.reviews/26152#l-194"

  q6="The `MiniMiner` is given a list of UTXOs (outpoints) that the wallet
      might be interested in spending. Given an outpoint, what are its five
      possible states?"
  a6="It could be (1) confirmed and unspent, (2) confirmed but already being spent
      by an existing transaction in the mempool, (3) unconfirmed (in the
      mempool) and unspent, (4) unconfirmed
      but already spent by an existing transaction in the mempool,
      or (5) it could be an outpoint that we've never heard of."
  a6link="https://bitcoincore.reviews/26152-2#l-21"

  q7="What approach is taken in the \"Bump unconfirmed parent txs to target
      feerate\" commit?"
  a7="This commit is the main behavior change of the PR.
      We use the `MiniMiner` to calculate the bump fees (the fees needed to bump
      their respective ancestries to the target feerate) of each UTXO and
      deduct those from their effective values.
      Then we run coin selection as before."
  a7link="https://bitcoincore.reviews/26152-2#l-100"

  q8="How does the PR handle spending unconfirmed UTXOs with overlapping ancestry?"
  a8="After coin selection, we run a variant of the `MiniMiner` algorithm
      on the _result_ of each coin selection run to get an exact bump fee.
      If we have over-bumped due to shared ancestry, we can reduce
      the fees by increasing the change value if it exists, or adding a
      change output if it doesn't exist."
  a8link="https://bitcoincore.reviews/26152-2#l-111"
%}

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

- [BTCPay Server 1.7.1][] is the latest release of the most widely used
  self-hosted payment processing software for Bitcoin. {% assign timestamp="54:04" %}

- [Core Lightning 22.11][] is the next major version of CLN.  It's also
  the first release to use a new version numbering scheme.[^semver]
  Included are several new features, including a new plugin manager,
  and multiple bug fixes. {% assign timestamp="55:21" %}

- [LND 0.15.5-beta][] is a maintenance release of LND.  It contains
  only minor bug fixes according to its release notes. {% assign timestamp="55:44" %}

- [BDK 0.25.0][] is a new release for this library for building wallets. {% assign timestamp="56:05" %}

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo], [Core
Lightning][core lightning repo], [Eclair][eclair repo], [LDK][ldk repo],
[LND][lnd repo], [libsecp256k1][libsecp256k1 repo], [Hardware Wallet
Interface (HWI)][hwi repo], [Rust Bitcoin][rust bitcoin repo], [BTCPay
Server][btcpay server repo], [BDK][bdk repo], [Bitcoin Improvement
Proposals (BIPs)][bips repo], and [Lightning BOLTs][bolts repo].*

- [Bitcoin Core #19762][] updates the RPC (and, by extension,
  `bitcoin-cli`) interface to allow named and positional arguments to be
used together. This change makes it more convenient to use named
parameter values without having to name every one.  The PR description
provides examples demonstrating the increased convenience of this
approach as well as a handy shell alias for frequent users of
`bitcoin-cli`. {% assign timestamp="57:42" %}

- [Core Lightning #5722][] adds [documentation][grpc doc] about how to
  use the GRPC interface plugin. {% assign timestamp="1:03:16" %}

- [Eclair #2513][] updates how it uses the Bitcoin Core wallet to ensure
  it always sends change to P2WPKH outputs.  This
  is the result of [Bitcoin Core #23789][] (see [Newsletter
  #181][news181 bcc23789]) where the project addressed a privacy
  concern for adopters of new output types (e.g. [taproot][topic
  taproot]).  Previously, a user who set their wallet default address
  type to taproot would also create taproot change outputs when they
  paid someone.  If they paid someone who didn't use taproot, it was
  easy for third parties to determine which output was the payment (the
  non-taproot output) and which was the change output (the taproot
  output).  After the change to Bitcoin Core, it would default to trying
  to use the same type of change output as the paid output, e.g. a
  payment to a native segwit output would also result in a native segwit
  change output.

  However, the LN protocol requires certain output types.  For
  example, a P2PKH output can't be used to open an LN channel.
  For that reason, users of Eclair with Bitcoin Core need to ensure
  they don't generate change outputs of an LN-incompatible type. {% assign timestamp="1:05:23" %}

- [Rust Bitcoin #1415][] begins using the [Kani Rust Verifier][] to
  prove some properties of Rust Bitcoin's code.  This complements other
  continuous integration tests performed on the code, such as fuzzing. {% assign timestamp="1:08:21" %}

- [BTCPay Server #4238][] adds an invoice refund endpoint to BTCPay's
  Greenfield API, a more recent API different from the original BitPay-inspired API. {% assign timestamp="1:09:16" %}

## Footnotes

[^semver]:
    Previous editions of this newsletter claimed Core Lightning used the
    [semantic versioning][] scheme and new versions would continue using
    that scheme in the future.  Rusty Russell has [described][rusty
    semver] why CLN can't completely adhere to that scheme.  We thank
    Matt Whitlock for notifying us about our previous error.

{% include references.md %}
{% include linkers/issues.md v=2 issues="19762,5722,2513,1415,4238,18725,26403,23789" %}
[lnd 0.15.5-beta]: https://github.com/lightningnetwork/lnd/releases/tag/v0.15.5-beta
[core lightning 22.11]: https://github.com/ElementsProject/lightning/releases/tag/v22.11
[btcpay server 1.7.1]: https://github.com/btcpayserver/btcpayserver/releases/tag/v1.7.1
[bdk 0.25.0]: https://github.com/bitcoindevkit/bdk/releases/tag/v0.25.0
[semantic versioning]: https://semver.org/spec/v2.0.0.html
[grpc doc]: https://github.com/cdecker/lightning/blob/20bc743840bf5d948efbf62d32a21a00ed233e31/plugins/grpc-plugin/README.md
[news181 bcc23789]: /en/newsletters/2022/01/05/#bitcoin-core-23789
[kani rust verifier]: https://github.com/model-checking/kani
[news223 anchors]: /en/newsletters/2022/10/26/#ephemeral-anchors
[news224 anchors]: /en/newsletters/2022/11/02/#anchor-outputs-workaround
[news220 v3tx]: /en/newsletters/2022/10/05/#proposed-new-transaction-relay-policies-designed-for-ln-penalty
[sanders ephemeral]: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-November/021222.html
[review club 26152]: https://bitcoincore.reviews/26152
[review club 26152-2]: https://bitcoincore.reviews/26152-2
[rusty semver]: https://github.com/ElementsProject/lightning/issues/5716#issuecomment-1322745630
