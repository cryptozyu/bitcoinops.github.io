---
title: Erlay

## Required.  At least one category to which this topic belongs.  See
## schema for options
topic-categories:
  - P2P Network Protocol
  - Bandwidth Reduction

## Required.  Use Markdown formatting.  Only one paragraph.  No links allowed.
excerpt: >
  **Erlay** is proposal to improve the bandwidth efficiency of relaying
  unconfirmed transactions between Bitcoin full nodes.

## Optional.  Produces a Markdown link with either "[title][]" or
## "[title](link)"
primary_sources:
    - title: BIP330

    - title: Erlay
      link: https://arxiv.org/pdf/1905.10518.pdf

## Optional.  Each entry requires "title", "url", and "date".  May also use "feature:
## true" to bold entry
optech_mentions:
  - title: Erlay proposed
    url: /en/newsletters/2019/06/05/#erlay-proposed

  - title: Draft BIP for enabling Erlay compatibility
    url: /en/newsletters/2019/10/02/#draft-bip-for-enabling-erlay-compatibility

  - title: Erlay-compatible transaction reconciliation protocol published as BIP330
    url: /en/newsletters/2019/11/13/#bips-851

  - title: "2019 year-in-review: erlay"
    url: /en/newsletters/2019/12/28/#erlay-and-other-p2p-improvements

  - title: "PR Review Club: #18261 implementing Erlay"
    url: /en/newsletters/2021/03/10/#bitcoin-core-pr-review-club

  - title: "PR Review Club: #23443 implementing Erlay support signaling"
    url: /en/newsletters/2022/01/12/#bitcoin-core-pr-review-club

  - title: "BIPs #1370 updates the BIP330 specification of Erlay"
    url: /en/newsletters/2022/10/05/#bips-1370

  - title: "Bitcoin Core #23443 adds the `sendtxrcncl` negotiation message for Erlay"
    url: /en/newsletters/2022/10/26/#bitcoin-core-23443

  - title: Discussion about inconsistent mempools and how that could affect Erlay
    url: /en/newsletters/2022/11/02/#better-peering-involves-tradeoffs

  - title: "Announcement of new Hyperion network simulator intended for eventual testing of Erlay"
    url: /en/newsletters/2024/08/02/#hyperion-network-event-simulator-for-the-bitcoin-p2p-network

  - title: "Erlay update and summary of results from multiple research questions"
    url: /en/newsletters/2025/02/07/#erlay-update

  - title: "Erlay may save 100 MB a day of node bandwidth, 20% of total bandwidth for some nodes"
    url: /en/newsletters/2025/03/14/#high-inv-traffic-as-a-non-archival-listener

## Optional.  Same format as "primary_sources" above
see_also:
  - title: Minisketch
    link: topic minisketch
---
In the currently-used Bitcoin gossip protocol, most full nodes are
configured to advertise every new transaction to all of their peers
unless they've previously received an advertisement about the
transaction from that peer.  At a minimum of 32 bytes per
advertised txid and nodes having a default maximum of 125 peers, this
consumes a large amount of redundant bandwidth given that each node
only needs to learn about a transaction from one of its peers.

Erlay is a two-part proposal that first limits the number of peers to
which a node will directly advertise transactions (default: 8) and,
second, uses set reconciliation based on [libminisketch][] with the
remainder of its peers to avoid sending the txid of any transactions
that the receiving peer has already seen.

Erlay scales to larger numbers of peers much better than the current
protocol, making it practical for nodes to accept more connections
than they do now.  This would improve the robustness of the relay
network against both accidental and deliberate network partitions.

{% include references.md %}
