---
layout: category
taxonomy: xbgp
title: "xBGP: making BGP truly extensible"
permalink: "/xbgp/"
category_list: xbgp
---

[BGP](https://tools.ietf.org/html/rfc4271) is one of the most important Internet routing protocols. In addition to its role to disseminate interdomain routes, BGP is also used inside ISP networks to support various value-added services including Virtual Private Networks. Large cloud provides have also started to use BGP as an intradomain routing protocol in [large datacenters](https://tools.ietf.org/html/rfc7938).

A BGP implementation is often considered as a blackbox that interacts
through different means:

 - it exchanges BGP messages over BGP sessions by sending them on TCP
 connections
 - it pushes the best
 - it interacts with the intradom routing protocols (e.g. OSPF or
 IS-IS) to extract the path and the cost to reach the BGP nexthops
 - it exposes various metrics using the BGP MIB
 - it can be configured using a proprietary Command Line Internet or
 protocols such as NETCONF

Such a BGP implementation is depicted in the figure below.

![BGP implementation]({{ site.baseurl }}/images/bgp.png)

BGP was designed more than thirty years ago. Since the first
publication of BGP-4, various extensions have been discussed and
approved within the
[IDR IETF Working Group](https://datatracker.ietf.org/wg/idr/about/).
These extensions have proposed significant changes to the
protocol. The figure below depicts the number of MUST/SHOULD
[RFC2119](https://tools.ietf.org/html/rfc2119) found in all the BGP
standards-track RFCs produced by the IDR working group during the last
two decades.

![Evolution of the BGP protocol]({{ site.baseurl }}/images/prot-evol-bgp.png)

Despite the importance of these protocol extensions, BGP
implementations have not been designed with extensibility in mind. In
[The case for pluginized routing protocols](https://inl.info.ucl.ac.be/publications/case-pluginized-routing-protocols.html),
we proposed a different model to organise one BGP implementation,
i.e. [FRRouting](https://frrouting.org). To enable network operators
to develop their own extension to the BGP protocol, we have added to
FRRouting a modified eBPF Virtual Machine that allows to execute
operator supplied bytecode at various points in the FRRouting code to
implement extensions to both OSPF and BGP. The figure below shows our
modified architecture.

![Pluginized FRRouting]({{ site.baseurl }}/images/bgp-plugins.png)

This modified BGP implementation contains an eBPF virtual machine that
executes plugins that are supplied by the network operators. This
approach allows to extend one particular implementation. In a
forthcoming paper, we demonstrate how a similar idea can be made more
generic and applied to different implementations so that a network
operator can write plugins which can be executed on different
implementations. This makes BGP implementations truly programmable.


<!-- Keep this last line here! -->
# Recent Posts