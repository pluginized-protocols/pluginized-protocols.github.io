---
layout: category
taxonomy: tcpls
title: "TCPLS : integrating TCP and TLS"
permalink: "/tcpls/"
category_list: tcpls
---

The Transmission Control Protocol ([TCP](https://tools.ietf.org/html/rfc793)) is one of the most important
Internet protocols. These days,
[TCP](https://tools.ietf.org/html/rfc793) is rarely used without
[TLS](https://tools.ietf.org/html/rfc8446).

TCP and TLS are independant protocols and TLS makes very few
assumptions on the service provided by the underlying TCP
implementation. This layering separation allows one TLS implementation
to be easily adapted to various TCP implementations

![TCP over TLS]({{ site.baseurl }}/images/tcp-tls.png)


TCPLS takes a different approach. Since both TCP and TLS are often
used together, we reconsider their interactions and design a
protocol that uses the features from one protocol to improve the
other. 

![TCPLS]({{ site.baseurl }}/images/tcpls.png)

TCPLS is a work in progress. We are continuously developing the
prototype and will publish new use cases and experiments in the coming
weeks on this website. 
