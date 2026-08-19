---
title: "Traffic filtering: ACLs and firewalls"
description: "From the stateless access list to the session table, and on to zone-based policy on IOS."
date: 2026-08-19
chapter: 4
translationKey: "psf-522-04"
weight: 40
tags: ["acl", "firewall", "zpf", "dmz"]
---

So far the target has been the device. Now it is the traffic crossing it, and the
question changes: instead of who gets into the router, who crosses the network and where
they are allowed to go.

## The access list and its four rules

An ACL is an ordered sequence of permit and deny statements the router evaluates against
each packet. Four behaviours explain nearly every lab mistake.

Evaluation runs top to bottom and stops at the first match. Wrong order produces an
unreachable rule, and the classic case denies a host after already permitting the whole
subnet.

Every ACL ends with an implicit deny you never see in `show`. A list holding one deny
statement blocks everything, because nothing permits the rest.

Matching uses a wildcard mask, the inverse of a network mask. Where a bit is zero, the
packet has to match. Where it is one, the router ignores it.

Direction matters. The same list applied inbound or outbound produces different effects,
because the decision happens before or after routing.

On placement, the traditional rule puts a standard ACL near the destination, since it
inspects only the source address and filtering early would drop legitimate traffic bound
elsewhere. An extended ACL goes near the source, since it distinguishes destination and
port, and dropping early saves bandwidth along the whole path.

## The return traffic problem

An ACL keeps no memory. Every packet is judged as if it were the first, which creates a
problem the moment you try to let the internal network reach the internet: the request
leaves, and the reply arrives on some arbitrary high port your list never anticipated.

The first fix permitted the return by reading TCP control bits. A packet belonging to an
established conversation carries flags the first packet does not, so the ACL demands
those flags. It works, and it is fragile: anyone forging the bits walks through, and UDP
carries no state to inspect at all.

The better answer was to keep state.

## The stateful firewall

A stateful firewall maintains a table of active connections. When a packet leaves, it
records source, destination, ports and the state of the conversation. When the reply
arrives, it compares against the table: the packet belongs to a session it watched
begin, so it passes. It does not, so it drops.

The practical difference is that you write the rule in one direction and the return
happens without you describing it. The conceptual difference is that the firewall now
understands conversations rather than isolated packets.

The families are worth placing, because exams ask. A packet filter reads headers and
nothing else. A stateful firewall adds the session table. An application gateway, or
proxy, terminates the connection on one side and opens another on the other, which
allows content inspection at the cost of performance and transparency. And NAT is not a
firewall, even though it hides internal topology: it translates addresses, and any
protection is a side effect rather than a design.

## Zones instead of interfaces

The older IOS model applied inspection interface by interface. It works, and it ages
badly: every new interface forces a review of the rules on all the others.

The zone model inverts the logic. You declare zones, put each interface in one, and
write policy between pairs of zones. Four behaviours define the model.

Traffic between two interfaces in the same zone passes with no policy. Traffic between
different zones is denied until a zone-pair with a policy permits it. Policy is
unidirectional, so permitting inside to outside permits nothing in the other direction.
And traffic between an interface in a zone and an interface in no zone gets dropped,
which produces the most common lab symptom: you configure everything, forget to place
one interface in a zone, and the network stops.

The router's own zone deserves a note. Traffic destined for the device itself, such as
your SSH session, belongs to a special zone that permits everything by default. The
moment you write a policy involving that zone, the default stops applying, and cutting
off your own administration becomes possible.

## The DMZ and why it exists

A server that has to be reachable from the internet cannot live on the internal network,
because compromising it would hand the attacker a foothold inside. It cannot live
outside either, because you lose control of it.

The DMZ is the third zone: reachable from outside through narrow rules, reachable from
inside, and forbidden from initiating connections inward. That last rule is the one that
matters. If the web server falls, the attacker sitting on it cannot open a session
against the internal database, because the policy denies that direction.

## Practice

Three interfaces on R1: `g0/0` for the internal network, `g0/1` for the internet, `g0/2`
for the DMZ with a web server at 192.168.100.10.

### Named extended ACL

```text
R1(config)# ip access-list extended EDGE-IN
R1(config-ext-nacl)# permit tcp any host 192.168.100.10 eq 80
R1(config-ext-nacl)# permit tcp any host 192.168.100.10 eq 443
R1(config-ext-nacl)# deny ip 10.0.0.0 0.255.255.255 any
R1(config-ext-nacl)# deny ip 127.0.0.0 0.255.255.255 any
R1(config-ext-nacl)# permit icmp any any echo-reply
R1(config-ext-nacl)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip access-group EDGE-IN in
```

The two deny lines in the middle drop private and loopback addresses arriving from
outside, which only exist in forged packets. Filtering that at the edge is basic hygiene.

### Stateless return

```text
R1(config)# ip access-list extended RETURN
R1(config-ext-nacl)# permit tcp any 10.10.0.0 0.0.255.255 established
```

Demonstrate this in the lab and then replace it, because the next block does the job
properly.

### Zone-based firewall

Declare the zones and what to inspect:

```text
R1(config)# zone security INSIDE
R1(config)# zone security OUTSIDE
R1(config)# zone security DMZ
R1(config)# class-map type inspect match-any CM-OUTBOUND
R1(config-cmap)# match protocol tcp
R1(config-cmap)# match protocol udp
R1(config-cmap)# match protocol icmp
R1(config-cmap)# exit
R1(config)# policy-map type inspect PM-INSIDE-OUTSIDE
R1(config-pmap)# class type inspect CM-OUTBOUND
R1(config-pmap-c)# inspect
R1(config-pmap-c)# exit
R1(config-pmap)# class class-default
R1(config-pmap-c)# drop log
```

Bind the interfaces and the zone-pair:

```text
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# zone-member security INSIDE
R1(config-if)# exit
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# zone-member security OUTSIDE
R1(config-if)# exit
R1(config)# interface gigabitEthernet 0/2
R1(config-if)# zone-member security DMZ
R1(config-if)# exit
R1(config)# zone-pair security ZP-IN-OUT source INSIDE destination OUTSIDE
R1(config-sec-zone-pair)# service-policy type inspect PM-INSIDE-OUTSIDE
```

To publish the DMZ server, its own zone-pair with a narrow policy:

```text
R1(config)# class-map type inspect match-any CM-WEB
R1(config-cmap)# match protocol http
R1(config-cmap)# match protocol https
R1(config-cmap)# exit
R1(config)# policy-map type inspect PM-OUT-DMZ
R1(config-pmap)# class type inspect CM-WEB
R1(config-pmap-c)# inspect
R1(config-pmap-c)# exit
R1(config)# zone-pair security ZP-OUT-DMZ source OUTSIDE destination DMZ
R1(config-sec-zone-pair)# service-policy type inspect PM-OUT-DMZ
```

Notice what was never written: no zone-pair with DMZ as source and INSIDE as
destination. That absence is the security rule, since the model denies by default.

### Checking

```text
R1# show access-lists
R1# show ip access-lists EDGE-IN
R1# show zone security
R1# show zone-pair security
R1# show policy-map type inspect zone-pair sessions
```

The last command shows the live session table. Generate traffic from inside to outside
and watch entries appear and expire, which is the most direct demonstration of the
difference between a stateless filter and a stateful firewall.

## Course labs

The subject appears in labs 10.3.11 and 10.3.12 of module 10, and in 4.4.1.2 from the
older CCNA Security material.

The next chapter deals with traffic that passed the filter and is malicious anyway.
