---
layout: category
taxonomy: tcp
title: "Towards more extensible TCP implementations"
permalink: "/tcp/"
category_list: tcp
---

[TCP](https://tools.ietf.org/html/rfc793) is one of the most important
protocols in today's Internet. TCP is used by a wide range of
applications that require a reliable data transfer.

A TCP implementation is usually considered as a blackbox that
interacts with other protocols through four main interfaces:

 - the socket API, or an equivalent, that enables applications to
 create and terminate connections but also send and receive data
 - an interface with the IP layer that enables TCP to send and receive
 packets and process ICMP messages
 - an interface with the SNMP agent that exposes a set of metrics
 through the TCP MIBs
 - a set of configuration knobs that allow to set system wide
 configuration parameters (e.g. systcl on Linux)


![TCP stack]({{ site.baseurl }}/images/tcp-stack.png)

TCP has evolved a lot since the publication of
[RFC793](https://tools.ietf.org/html/rfc793). The latest TCP roadmap
document, [RFC7414](https://tools.ietf.org/html/rfc7414) summarises
all the standards-track and experimental RFCs that extend
TCP. Extending TCP implementations requires time and
effort. Measurement studies demonstrate that several TCP
implementations took many years to support new extensions.
In
[An Analysis of Longitudinal TCP Passive Measurements](https://link.springer.com/chapter/10.1007/978-3-642-20305-3_3
) shows that it took more than a decade to fully deploy of
Selective Acknowledgments, defined in
[RFC2018](https://tools.ietf.org/html/rfc2018) and Window Scale
defined in [RFC1323](https://tools.ietf.org/html/rfc1323). The
TCP Timestamps option, also defined in
[RFC1323](https://tools.ietf.org/html/rfc1323) is still not supported
by the Windows stack. In
[Tracking transport-layer evolution with PATHspider](https://irtf.org/anrw/2017/anrw17-final16.pdf),
Trammel et al. show that TCP Fast Open (TFO)
[RFC7413](https://tools.ietf.org/html/rfc7413) is still not widely
deployed. The same applies to Multipath TCP
[RFC6824](https://tools.ietf.org/html/rfc6824).

The development and the deployment of these extensions shows that
future protocol extensions need to be taken into account by
implementors when designing an implementation. What could implementors
do to better support the extensions that will be invented for the
protocol that they implement ?

We believe that one way to solve this problem is to make the protocol
implementations extensible by designing. This implies that in addition
to the four interfaces mentioned above, a TCP implementation should
also include an `eXtensibility Interface` (XI) that enables it to be
extended by using extensions that we call plugins. A plugin is a piece
of platform independent executable code which can be attached to an
existing TCP implementation to extend it.

There are probably different ways to realize this `eXtensibility
Interface` in different protocols. For the Linux TCP stack, such an
interface can be created by leveraging the recent efforts that added
eBPF to this stack. Recent versions of the Linux kernel include an
eBPF virtual machine that allows to execute user-supplied bytecode to
collect information about the kernel, but also execute specific
algorithms. Several hooks exist to expose some functions of the Linux
TCP stack to eBPF bytecode. We have demonstrated how
to use this facility to extend the Linux TCP stack in recent papers:

 - [Viet-Hoang-Tran](https://inl.info.ucl.ac.be/hoang.html), [Olivier Bonaventure](https://perso.uclouvain.be/olivier.bonaventure), [Making the Linux TCP stack more extensible with eBPF](https://inl.info.ucl.ac.be/system/files/tcp-ebpf.pdf),
   Netdev 0x03
 - [Viet-Hoang-Tran](https://inl.info.ucl.ac.be/hoang.html), [Olivier Bonaventure](https://perso.uclouvain.be/olivier.bonaventure), [Beyond socket options: making the Linux TCP stack truly extensible](https://inl.info.ucl.ac.be/system/files/paper-bpf.pdf),
   IFIP Networking 2019
 - [Viet-Hoang-Tran](https://inl.info.ucl.ac.be/hoang.html),
   [Olivier Bonaventure](https://perso.uclouvain.be/olivier.bonaventure),
   [Beyond socket options: Towards fully extensible Linux transport stacks](https://dial.uclouvain.be/pr/boreal/object/boreal%3A235010/datastream/PDF_01/view),Computer
   Communications, 2020

The idea of using eBPF to support the implementation of new TCP
   options using eBPF bytecode has been adopted by the Linux kernel
   implementors with
   [patches](https://lore.kernel.org/netdev/20200626175501.1459961-1-kafai@fb.com/)
   proposed by Martin Kai Lau.

The next step will be to make this idea portable over different TCP implementations.





