---
title: 'Bitcoin Optech Newsletter #154'
permalink: /en/newsletters/2021/06/23/
name: 2021-06-23-newsletter
slug: 2021-06-23-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter describes a proposal to allow universal
transaction replacement by fee and includes the first post in a new
weekly series about preparing for taproot.  Also included are our
regular sections describing updates to clients and services, new releases
and release candidates, and notable changes to popular Bitcoin
infrastructure projects.

## News

- **Allowing transaction replacement by default:** almost all Bitcoin
  full nodes today are believed to implement [BIP125][] opt-in Replace
  By Fee ([RBF][topic rbf]), which allows unconfirmed transactions to be
  replaced in node mempools by alternative versions that pay higher
  fees---but only if the creator of the transaction sets a signal in the
  original transaction.  This opt-in behavior was proposed as a
  compromise between people who wanted to allow transaction replacement,
  such as for fee bumping or [additive payment batching][], and people
  who objected because allowing replacement simplifies building tools
  that defraud merchants who accept unconfirmed transactions as final.

  Over five years later, it appears very few merchants today are
  accepting unconfirmed transactions as final, and it's not clear how
  many of those that do are actually checking for the BIP125 opt-in
  signal and treating those transactions differently.  If no one is
  relying on BIP125 signals, then allowing every transaction to be
  replaceable could provide some advantages, such as:

  - **Simplifying analysis** for presigned transaction protocols (such
    as LN and [vaults][topic vaults]) where ideas for using RBF fee
    bumping need to account for a malicious counterparty's ability to
    prevent setting the BIP125 signal.  If every transaction could be
    replaced, this wouldn't be a concern.

  - **Reducing transaction analysis opportunity** because transactions
    that opt in to RBF look different onchain than transactions which
    don't.  Since most wallets consistently opt in, or not, this
    provides evidence that surveillance companies can use in their
    attempts to identify who owns which bitcoins.  If every
    transaction was replaceable, there'd be no need to set the BIP125
    signal.

  This week, Antoine Riard [posted][riard rbf] a proposal to the Bitcoin-Dev
  mailing list for eventually changing Bitcoin Core's code to allow
  RBF for all transactions regardless of whether or not they set the
  BIP125 opt-in signal.  The idea was also discussed in the first
  transaction relay workshop [meeting][trw meeting].  Several meeting
  participants mentioned Bitcoin Core [PR #10823][bitcoin core #10823]
  as an alternative approach---it allows any transaction to be
  replaced, but only after the transaction had spent a certain amount
  of time in a node mempool (originally proposed as 6 hours; later
  suggested to be 72 hours).

  Both Riard's email and the meeting participants note that any proposal
  for replacing transactions that don't contain a BIP125 opt-in signal
  requires feedback from merchants currently depending on
  BIP125 behavior.  Optech encourages any such merchants to
  respond to the mailing list thread.

## Changes to services and client software

*In this monthly feature, we highlight interesting updates to Bitcoin
wallets and services.*

- **Trezor Suite adds RBF support:**
  Trezor's wallet software, Trezor Suite, added [support for Replace-by-Fee
  (RBF)][trezor rbf] in version 21.2.2. RBF is on by default and also supported
  by some of Trezor's hardware devices.

- **Lightning Labs announces Terminal Web:**
  In a recent [blog post][lightning labs terminal web blog], Lightning Labs
  describes their web-based Lightning node scoring dashboard, [Terminal
  Web][terminal web].

- **Specter v1.4.0 released:**
  [Specter v.1.4.0][specter v1.4.0] adds a feature to ["cancel" a
  transaction][specter 1197] using BIP125 opt-in [Replace-by-Fee (RBF)][topic
  rbf].

- **Phoenix adds LNURL-pay:**
  ACINQ's mobile wallet [Phoenix][phoenix wallet] added support for the
  [LNURL-pay][lnurl-pay github] protocol in its [v1.4.12 release][phoenix
  1.4.12].

- **JoinMarket v0.8.3 released:**
  [JoinMarket v0.8.3][joinmarket v0.8.3] adds the ability to provide custom
  change addresses and an Electrum-compatible segwit `signmessage`
  implementation.

## Preparing for taproot #1: bech32m sending support

*The first segment in a weekly [series][series preparing for taproot] about how developers and service providers
can prepare for the upcoming activation of taproot at block height
{{site.trb}}.*

{% include specials/taproot/00-bech32m.md %}

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

- [LND 0.13.0-beta][] is a new major release that improves feerate
  management by making [anchor outputs][topic anchor outputs] the
  default commitment transaction format, adds support for using a pruned
  Bitcoin full node, allows receiving and sending payments using Atomic
  MultiPath ([AMP][topic multipath payments]), and increases LND's
  [PSBT][topic psbt] capabilities, among many other improvements and bug
  fixes.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[Rust-Lightning][rust-lightning repo], [libsecp256k1][libsecp256k1
repo], [Hardware Wallet Interface (HWI)][hwi repo],
[Rust Bitcoin][rust bitcoin repo], [BTCPay Server][btcpay server repo],
[Bitcoin Improvement Proposals (BIPs)][bips repo], and [Lightning
BOLTs][bolts repo].*

- [Bitcoin Core #21365][] adds the ability for the wallet to create
  signatures for [taproot][topic taproot] spends---both keypath spends
  using only the P2TR public key and scriptpath spends using a
  [tapscript][topic tapscript].  The wallet can also sign for
  taproot-spending [PSBTs][topic psbt], but only if the wallet already
  has all the keypath or scriptpath information it needs.  The somewhat
  related merged PR [#22156][bitcoin core #22156] only allows importing
  that keypath and scriptpath information after taproot is active (block
  {{site.trb}} on mainnet, but on test networks where taproot is already
  enabled, importing may be used now).

- [Bitcoin Core #22144][] randomizes the order in which peers are serviced
  in the message handling thread, which is
  responsible for parsing and processing P2P messages from peers and for
  sending messages to those peers. Previously, the message handling thread
  would service each peer round-robin _in the order in which the
  connections to those peers were first established_. This PR changes the
  logic so that, on each iteration of the message handling loop, the order in
  which peers are serviced is randomized. Peers are still serviced with the same
  frequency (each peer is serviced once per iteration), but any weaknesses or
  exploits that rely on a deterministic ordering of servicing peers are
  avoided.

- [Bitcoin Core #21261][] makes it easier to extend inbound connection
  protection to more networks and then uses that framework to
  add [I2P][] to the list of protected networks.  Diversity
  protection (often called eviction protection) allows a few peers with
  desirable characteristics to remain connected when Bitcoin Core is
  otherwise pruning high-latency connections.  Retaining a few
  connections to peers on anonymity networks is highly desirable both
  because it allows transaction creators to use those networks to hide
  their network identity and because the ability to receive blocks over
  those networks in addition to the regular Internet Protocol can
  prevent some types of [eclipse attacks][topic eclipse attacks].

- [Rust Bitcoin #601][] adds support for parsing [bech32m][topic bech32]
  addresses and requires that v1+ native segwit addresses be encoded with
  bech32m and not bech32.

- [BTCPay Server #2450][] makes generating [payjoin][topic
  payjoin]-compatible invoices the default when the user opts into using
  a hot wallet for receiving payments.  A button on the *create wallet*
  screen allows the user to opt out of this default setting.

- [BTCPay Server #2559][] adds a separate screen for guiding the user
  through their choices for how to sign transactions they spend from their
  wallet.  For hot wallets, the server can just sign, but for wallets
  where the keys are stored elsewhere, an attractive and informative GUI
  now guides the user through signing options such as entering their
  recovery mnemonic, using a hardware signing device, or generating a
  PSBT for transfer to a signing wallet.

{% include references.md %}
{% include linkers/issues.md issues="10823,21365,22156,22144,21261,601,2450,2559" %}
[LND 0.13.0-beta]: https://github.com/lightningnetwork/lnd/releases/tag/v0.13.0-beta.rc5
[trw meeting]: https://gist.githubusercontent.com/ariard/5f28dffe82ddad763b346a2344092ba4/raw/2a8e0d4ff431a225a970d0128aa78616df6b6382/meeting-logs
[additive payment batching]: /en/cardcoins-rbf-batching/
[riard rbf]: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-June/019074.html
[i2p]: https://en.wikipedia.org/wiki/I2P
[phoenix wallet]: https://phoenix.acinq.co/
[phoenix 1.4.12]: https://github.com/ACINQ/phoenix/releases/tag/v1.4.12
[lnurl-pay github]: https://github.com/fiatjaf/lnurl-rfc/blob/master/lnurl-pay.md
[joinmarket v0.8.3]: https://github.com/JoinMarket-Org/joinmarket-clientserver/releases/tag/v0.8.3
[specter v1.4.0]: https://github.com/cryptoadvance/specter-desktop/releases/tag/v1.4.0
[specter 1197]: https://github.com/cryptoadvance/specter-desktop/pull/1197
[terminal web]: https://terminal.lightning.engineering/
[lightning labs terminal web blog]: https://lightning.engineering/posts/2021-05-11-terminal-web/
[trezor rbf]: https://wiki.trezor.io/Replace-by-fee_(RBF)
