---
title: 'Bitcoin Optech Newsletter #107'
permalink: /en/newsletters/2020/07/22/
name: 2020-07-22-newsletter
slug: 2020-07-22-newsletter
type: newsletter
layout: newsletter
lang: en
---
This week's newsletter links to several discussions about activating
taproot and summarizes a proposed update to BIP173 bech32 addresses.
Also included are our regular sections summarizing interesting changes to
services and client software, releases and release candidates, and notable
changes to popular Bitcoin infrastructure software.

## Action items

*None this week.*

## News

- **Taproot activation discussions:** several discussions were started
  or continued this week about choosing a method for activating
  [taproot][topic taproot].

  - **New IRC room:** Steve Lee [posted][lee irc] to the Bitcoin-Dev
    mailing list an announcement of a new `##taproot-activation` IRC
    room on the [Freenode][] network.  The channel is [logged][irc
    log] and saw an impressive amount of conversation from dozens of
    participants during its first week.

  - **Mailing list thread:** Anthony Towns [posted][towns post] a summary of
    the recently updated [BIP8][] activation method (see [Newsletter
    #104][news104 bip8]) and of his own new [bip-decthresh][]
    activation proposal (based on Matt Corallo's "modern soft fork
    activation" [post][msfa] in January, see [Newsletter #80][news80
    activation]).  An interesting feature of the decreasing threshold
    idea is that the amount of network hashrate needed to signal
    readiness to enforce the new soft fork rules would decrease during
    the final activation period, allowing miner activation with
    perhaps as little as 60% miner support rather than the 95% miner
    support needed during the first activation period.

- **Bech32 address updates:** Russell O'Connor [posted][oconnor post] to
  the Bitcoin-Dev mailing list a reply to Pieter Wuille's November post
  (see [Newsletter #77][news77 bech32]) about amending [BIP173][] to only
  allow witness programs of either 20 or 32 bytes, minimizing the risk
  that typos would cause a user to irrecoverably send bitcoins to a
  wrong address.  O'Connor made the counter-proposal that we could allow
  any witness program that had five more characters in its address than
  the next shortest address, up to the consensus-enforced length limit
  on witness programs.  The counter-proposal didn't receive any
  discussion on the mailing list, but a [PR][BIPs #945] to update BIP173
  did receive several comments.  As of this writing, no conclusion has
  been reached about what length restrictions to use.

## Changes to services and client software

*In this monthly feature, we highlight interesting updates to Bitcoin
wallets and services.*

- **Electrum adds Lightning Network and PSBT support:**
  [Electrum's 4.0.1][electrum release notes] LN support includes local and
  remote [watchtower][topic watchtowers] capabilities
  as well as submarine swaps using Electrum Technologies' central server.
  Electrum has also replaced their partial transaction format with PSBT.

- **Lily Wallet initial release:**
  [Lily Wallet][lily wallet site] has announced the initial v0.0.1-beta release
  of their multi-account desktop multisig vault application. Lily Wallet creates
  2-of-3 bech32 addresses and uses [HWI][topic hwi], HD wallets, and PSBT to transact.

- **Zap 0.7.0 Beta released:** [Zap 0.7.0 Beta][zap 0.7.0] adds [spontaneous payments][topic
  spontaneous payments], [payment probes][news30 probing], [multipath
  payments][topic multipath payments], and [hold invoices][topic hold invoices].

- **BTCPay Server 1.0.5.0 implements various standards:**
  [BTCPay Server 1.0.5.0][btcpay 1.0.5.0] adds [BIP78][], [BIP21][], and additional [PSBT][topic psbt]
  support.

## Releases and release candidates

*New releases and release candidates for popular Bitcoin infrastructure
projects.  Please consider upgrading to new releases or helping to test
release candidates.*

- [LND 0.10.4][] is a minor release that fixes a bug affecting backups
  on Windows.

- [C-Lightning 0.9.0rc2][C-Lightning 0.9.0] is a release candidate for
  an upcoming major release.  It adds support for the updated `pay`
  command and new `keysend` RPC described in this newsletter's *notable
  code changes* section, among several other notable changes and
  multiple bug fixes.

## Notable code and documentation changes

*Notable changes this week in [Bitcoin Core][bitcoin core repo],
[C-Lightning][c-lightning repo], [Eclair][eclair repo], [LND][lnd repo],
[Rust-Lightning][rust-lightning repo], [libsecp256k1][libsecp256k1 repo],
[Hardware Wallet Interface (HWI)][hwi], [Bitcoin Improvement Proposals
(BIPs)][bips repo], and [Lightning BOLTs][bolts repo].*

- [Bitcoin Core #19109][] introduces further improvements to
  [transaction-origin privacy][transaction origin wiki]. Building on
  [Bitcoin Core #18861][bitcoin 18861 newsletter], this patch adds a
  per-peer rolling bloom filter to track which transactions were recently
  announced to the peer. When a peer requests a transaction, this upgrade
  checks the filter before fulfilling the request and relaying the
  transaction to prevent spy nodes from learning about the exact contents
  of our mempool or the precise timing of when a transaction was received.

- [Bitcoin Core #16525][] updates several RPCs to return transaction
  version numbers as unsigned integers rather than signed integers.
  Early versions of Bitcoin interpreted version numbers as signed
  integers (and
  [some][35e79ee733fad376e76d16d1f10088273c2f4c2eaba1374a837378a88e530005]
  transactions in the chain did actually set negative version numbers),
  but the introduction of [BIP68][] nSequence enforcement for
  transactions with version 2 or higher specified that it would match
  against transaction nVersion fields cast to unsigned integers.  It's
  expected that any other future soft forks that use transaction
  versions will specify the same rule, and so it makes sense for RPCs to
  always return unsigned numbers for transaction versions.

- [C-Lightning #3809][] adds support to C-Lightning for effective
  sending of [multipath payments][topic multipath payments]---payments
  which are split into several parts, with each part routed using a
  different path.  In brief, the algorithm C-Lightning uses splits
  a payment into parts of approximately 0.0001 BTC in value (each part having its
  amount randomly fuzzed by plus or minus 10%).  If any sent part fails,
  that part is split into two parts (roughly in half, plus or minus 10%)
  and the two parts are re-sent.  The PR additionally adds a
  `disable-mpp` configuration option that will prevent sending any
  multipath payments; a parameter of the same name is also added to the
  `pay` command to disable sending a multipath payment for that
  particular attempt.

- [C-Lightning #3825][] changes the [recently merged][news105 cl3775]
  (but never released) `reserveinputs` RPC so that it no longer
  generates a new [PSBT][topic psbt].  Instead, a new `fundpsbt` RPC is
  introduced to create a new PSBT.  An existing PSBT may now be passed
  to `reserveinputs` to prevent the UTXO inputs to the PSBT from being
  used in other transactions until a specified block height (default, 72
  blocks in the future, about 12 hours from present) or as long as the
  daemon is running.  Alternatively, `fundpsbt` will default to
  reserving the selected UTXOs when it creates the transaction.

- [C-Lightning #3826][] completes a series of PRs that modify the logic
  C-Lightning uses to send payments.  Most of those changes should be
  invisible to users, but in case of problems, the previous
  logic available with the `pay` command is available using the new
  `legacypay` command.  Anyone using the `pay` command from this point
  forward will be using the new logic.

- [C-Lightning #3792][] adds a new `keysend` RPC for sending
  [spontaneous payments][topic spontaneous payments].  This new RPC is different
  from the `keysend` plugin, described in [Newsletter #94][news94
  keysend], which allows C-Lightning to *receive* spontaneous payments.

- [LND #4429][] adds a `--protocol.wumbo` configuration option and
  enables it by default.  When supported by both the local node and a
  remote peer, this option allows the opening of [large channels][topic
  large channels] where the total channel value exceeds 0.16 BTC.

{% include references.md %}
{% include linkers/issues.md issues="19109,16525,3809,3825,3826,3792,4429,945" %}
[C-Lightning 0.9.0]: https://github.com/ElementsProject/lightning/releases/tag/v0.9.0rc2
[LND 0.10.4]: https://github.com/lightningnetwork/lnd/releases/tag/v0.10.4-beta
[bitcoin 18861 newsletter]: /en/newsletters/2020/05/27/#bitcoin-core-18861
[transaction origin wiki]: https://en.bitcoin.it/wiki/Privacy#Traffic_analysis
[news105 cl3775]: /en/newsletters/2020/07/08/#c-lightning-3775
[35e79ee733fad376e76d16d1f10088273c2f4c2eaba1374a837378a88e530005]: https://blockstream.info/tx/35e79ee733fad376e76d16d1f10088273c2f4c2eaba1374a837378a88e530005
[lee irc]: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-July/018042.html
[irc log]: http://gnusha.org/taproot-activation/
[news104 bip8]: /en/newsletters/2020/07/01/#bips-550
[bip-decthresh]: https://github.com/ajtowns/bips/blob/202007-activation-dec-thresh/bip-decthresh.mediawiki
[msfa]: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-January/017547.html
[news80 activation]: /en/newsletters/2020/01/15/#discussion-of-soft-fork-activation-mechanisms
[oconnor post]: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-July/018048.html
[news77 bech32]: /en/newsletters/2019/12/18/#review-bech32-action-plan
[news94 keysend]: /en/newsletters/2020/04/22/#c-lightning-3611
[freenode]: https://freenode.net/
[towns post]: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-July/018043.html
[electrum release notes]: https://github.com/spesmilo/electrum/blob/4.0.1/RELEASE-NOTES
[lily wallet site]: http://lily.kevinmulcrone.com/
[news30 probing]: /en/newsletters/2019/01/22/#eclair-762
[zap 0.7.0]: https://github.com/LN-Zap/zap-desktop/releases/tag/v0.7.0-beta
[btcpay 1.0.5.0]: https://blog.btcpayserver.org/btcpay-server-1-0-5-0/
[hwi]: https://github.com/bitcoin-core/HWI
