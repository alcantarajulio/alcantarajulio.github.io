---
title: "Site-to-site IPsec VPNs"
description: "What IPsec is, what IKE negotiates in each phase, and how to build a tunnel between two routers from the CLI."
date: 2026-08-19
chapter: 7
translationKey: "psf-522-07"
weight: 70
tags: ["vpn", "ipsec", "ike", "isakmp"]
---

Two sites of the same organisation, one in Campina Grande and one in Coventry, need to
exchange internal traffic. Leasing a dedicated circuit between them works and costs a
fortune. A VPN uses the internet as transport and rebuilds in software the privacy the
dedicated circuit gave through physical construction.

## Two ways of using one

A site-to-site VPN joins two networks. The edge routers do all the work, the computers
on either side never learn a tunnel exists, and traffic between the subnets crosses
encrypted without anyone configuring the machines.

A remote access VPN joins one person to a network. Client software on the user's machine
builds the tunnel, and the design changes because the source address varies with every
connection.

This chapter covers the first, which is the one the course configures from the CLI.

## IPsec is a framework, not an algorithm

That distinction catches people in exams. IPsec defines which security services exist
and how to negotiate them, and leaves the choice of algorithm open. That allows
replacing a broken cipher without replacing the protocol, which has already happened
more than once.

It offers four services. Confidentiality through encryption of the payload. Integrity
through a keyed digest. Origin authentication. And replay protection, which discards a
valid packet captured and resent by an attacker.

Two headers implement this. The first authenticates without encrypting, which makes it
of little use today and incompatible with address translation, since it protects fields
NAT rewrites. The second encrypts and authenticates, and it is the one people use.

There are also two modes. In transport mode the original IP header stays and only the
payload is protected, which suits host-to-host communication. In tunnel mode the entire
packet goes inside a new one, which hides internal addresses and is what a site-to-site
VPN uses.

## What IKE negotiates

The two routers have to agree on algorithms, authenticate each other, and derive keys
before protecting any data. The protocol that does this works in two phases, and
understanding the split explains nearly every tunnel that refuses to come up.

The first phase establishes a secure channel for the negotiation itself. The peers agree
on the cipher, the digest algorithm, the authentication method, the Diffie-Hellman group
and the lifetime, run the key exchange, and prove identity to each other with a shared
secret or a certificate. The result is one bidirectional security association.

The second phase uses that channel to negotiate protection for the data. This is where
the transform set and the traffic the tunnel will carry get decided. The result is two
unidirectional associations, one per direction. You can enable perfect forward secrecy,
which runs a fresh Diffie-Hellman exchange in phase 2, so compromising the long-term key
decrypts no past session.

On versions, the first generation of IKE is the one the course configures and the one
still found on older equipment. The second generates fewer messages, handles NAT and
mobility better, and is the correct choice for new designs.

## The five steps

The lab reads more clearly once you see the sequence.

First, define interesting traffic, the access list describing what belongs in the
tunnel. Second, negotiate phase 1. Third, negotiate phase 2. Fourth, transfer protected
data. Fifth, tear down when the lifetime expires or the traffic stops.

The first step carries a trap. The interesting traffic list has to mirror between the
two sides: what one describes as source, the other describes as destination. Lists that
fail to mirror make phase 2 fail with an obscure message, and that is the most common
defect in the lab.

## NAT and the order of operations

The router applies NAT before applying encryption on egress. If your NAT rule translates
the traffic that should enter the tunnel, it leaves translated, stops matching the
interesting traffic list, and the tunnel never comes up. The fix exempts traffic bound
for the far end from NAT, and that exemption is the second most common defect.

## Practice

R1 at 200.1.1.1 and R3 at 200.3.3.3, each with an internal network: 10.10.0.0/24 behind
R1 and 10.30.0.0/24 behind R3.

### Phase 1

```text
R1(config)# crypto isakmp policy 10
R1(config-isakmp)# encryption aes 256
R1(config-isakmp)# hash sha256
R1(config-isakmp)# authentication pre-share
R1(config-isakmp)# group 14
R1(config-isakmp)# lifetime 3600
R1(config-isakmp)# exit
R1(config)# crypto isakmp key Vpn#Ufcg2026 address 200.3.3.3
```

Group 14 uses 2048 bits. Groups 1, 2 and 5 turn up in older material and belong in no
new design.

### Interesting traffic

```text
R1(config)# ip access-list extended VPN-TRAFFIC
R1(config-ext-nacl)# permit ip 10.10.0.0 0.0.0.255 10.30.0.0 0.0.0.255
```

On R3, the same list with source and destination swapped.

### Phase 2 and the crypto map

```text
R1(config)# crypto ipsec transform-set TS-AES esp-aes 256 esp-sha256-hmac
R1(cfg-crypto-trans)# mode tunnel
R1(cfg-crypto-trans)# exit
R1(config)# crypto map CMAP 10 ipsec-isakmp
R1(config-crypto-map)# set peer 200.3.3.3
R1(config-crypto-map)# set transform-set TS-AES
R1(config-crypto-map)# set pfs group14
R1(config-crypto-map)# set security-association lifetime seconds 1800
R1(config-crypto-map)# match address VPN-TRAFFIC
R1(config-crypto-map)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# crypto map CMAP
```

The map goes on the egress interface, the one facing the internet. Applying it to the
internal interface is the third classic mistake.

### Exempting the tunnel from NAT

```text
R1(config)# ip access-list extended NAT-OUT
R1(config-ext-nacl)# deny ip 10.10.0.0 0.0.0.255 10.30.0.0 0.0.0.255
R1(config-ext-nacl)# permit ip 10.10.0.0 0.0.0.255 any
R1(config-ext-nacl)# exit
R1(config)# ip nat inside source list NAT-OUT interface gigabitEthernet 0/1 overload
```

The deny line comes first and is the exemption. Without it, tunnel traffic leaves
translated and the tunnel stays down.

### Bringing it up and checking

The tunnel is born only when traffic matching the list appears. Generate some:

```text
PC1> ping 10.30.0.10
```

Then verify on both sides:

```text
R1# show crypto isakmp sa
R1# show crypto ipsec sa
R1# show crypto map
R1# show crypto session detail
```

`show crypto isakmp sa` has to report phase 1 as active. If it comes back empty, the
problem lives in phase 1: mismatched policy, different key, or the wrong peer address.
If it shows active and `show crypto ipsec sa` counts no encrypted packets, the problem
lives in phase 2, and the prime suspect is an unmirrored traffic list.

The counters in `show crypto ipsec sa` are the final test. Encapsulated packets climbing
on one side and decapsulated packets climbing on the other prove the tunnel carries real
data. A frozen counter with an established association means a tunnel nobody uses, which
usually turns out to be routing or NAT.

## Course labs

The subject appears in labs 19.5.5 and 19.5.6 of module 19.

The last chapter closes the guide by testing everything the previous seven configured.
