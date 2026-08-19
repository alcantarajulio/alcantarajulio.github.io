---
title: "Hardening and monitoring"
description: "Switching off what nobody uses, authenticating routing, synchronising the clock, and making logs land somewhere."
date: 2026-08-19
chapter: 2
translationKey: "psf-522-02"
weight: 20
tags: ["hardening", "ospf", "ntp", "syslog", "snmp"]
---

The previous chapter locked the front door. That leaves the rest of the device: a dozen
services Cisco enables at the factory for convenience, a routing protocol that accepts
any neighbour who introduces itself, a clock that drifts a few seconds a day, and an
event log that dies with the device when it reboots.

## Attack surface is the sum of what is switched on

Every active service is a door somebody can knock on, an implementation that may carry a
flaw, and one more line on your list of things to watch. A factory-default router answers
requests nobody has used in twenty years.

CDP announces the model, the IOS version, the management address and the device name to
anyone on the segment. For the engineer documenting the network that saves hours. For
someone doing reconnaissance before an attack, it saves the same hours. The built-in HTTP
server accepts clear-text authentication. The small servers answer echo, discard and
chargen, useful for diagnostics in 1983 and useful today as an amplifier in a denial of
service attack. Source routing lets the sender choose the path, which routes around the
filtering topology you built.

The rule is easy to state and tedious to apply: switch everything off, then switch back
on only what you use and can justify.

## The control plane believes whoever introduces itself

A routing protocol without authentication forms an adjacency with any neighbour speaking
the same language in the same area. Plug a device into the segment, advertise a better
route to the server subnet, and the traffic arrives. This is route hijacking, and the
result ranges from a black hole to silent interception, since nothing stops the attacker
forwarding the traffic onward after reading it.

Neighbour authentication closes the case. Both sides share a key, every message carries a
digest computed with it, and the router discards anything that fails the check. Older
implementations use MD5; current ones accept key chains with SHA, which also allow
rotation without dropping the adjacency.

## No shared clock, no investigation

An event log without trustworthy timestamps is an anecdote. When the firewall says 14:03,
the router says 13:58 and the server says 14:11, nobody reconstructs the order of events,
and the order of events is what separates cause from consequence.

NTP solves synchronisation and creates a new problem, because an attacker controlling the
time source rewrites the timeline of your investigation, makes an expired certificate look
valid, and shifts lockout windows out of alignment. So NTP authenticates too: the client
accepts a reply only from a server that proves it knows the key.

Two choices come with this. Log in UTC or mark the timezone explicitly, because
correlating logs from three countries in local time with no timezone costs an afternoon.
And timestamp with milliseconds, since automated events land inside the same second.

## A log that stays on the device is not a log

The local buffer disappears on reboot, and rebooting or clearing is the first move of
whoever gets in. Logs exist for later, so they have to leave the device while they still
exist.

Syslog sorts every message into eight levels, from 0 for emergency to 7 for debugging.
Sending everything at level 7 to the server produces volume nobody reads and buries what
matters. Sending only level 0 hides the login attempt that precedes the incident. The
balance usually sits at informational, level 6, with debugging switched on only during an
investigation.

SNMP deserves the same scrutiny. Versions 1 and 2c authenticate with a community string
that travels in clear text and works as a shared password granting read or write access
over the device. Version 3 adds per-user authentication and encrypts the payload. No
defensible reason exists for SNMP v2c on a new network.

## Getting the device back after the damage

IOS keeps a protected copy of the image and the configuration, hidden from the directory
listing and resistant to deletion by someone holding full privilege. This does not prevent
the intrusion. It shortens the gap between the damage and the return of service, which is
a metric your manager understands.

There is also a one-shot assistant that applies a large set of these measures at once. It
works well in a lab and on a new device. In production, read what it changed before
saving, because it disables services your network may depend on.

## Practice

Same scenario as the previous chapter, now with two routers running OSPF in area 0 and a
syslog and NTP server on the management range.

### Switching services off

```text
R1(config)# no cdp run
R1(config)# no ip http server
R1(config)# ip http secure-server
R1(config)# no service pad
R1(config)# no ip bootp server
R1(config)# no ip source-route
R1(config)# no service tcp-small-servers
R1(config)# no service udp-small-servers
R1(config)# no ip domain-lookup
```

On outward-facing interfaces, switch off what helps reconnaissance and redirection:

```text
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# no ip proxy-arp
R1(config-if)# no ip redirects
R1(config-if)# no ip unreachables
R1(config-if)# no ip mask-reply
```

If you need CDP somewhere, disable it per interface rather than keeping it global:

```text
R1(config-if)# no cdp enable
```

### Authenticating OSPF

```text
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip ospf message-digest-key 1 md5 Ospf#Redes2026
R1(config-if)# exit
R1(config)# router ospf 1
R1(config-router)# area 0 authentication message-digest
```

Apply this on both sides. A mismatched key drops the adjacency, and the symptom shows up
in `show ip ospf neighbor` as a neighbour stuck in INIT.

### Synchronising time with authentication

```text
R1(config)# ntp authentication-key 1 md5 Ntp#Redes2026
R1(config)# ntp authenticate
R1(config)# ntp trusted-key 1
R1(config)# ntp server 10.20.0.10 key 1
R1(config)# ntp update-calendar
R1(config)# clock timezone BRT -3
```

### Logging off the box

```text
R1(config)# service timestamps log datetime msec localtime show-timezone
R1(config)# service timestamps debug datetime msec
R1(config)# logging buffered 16384 informational
R1(config)# logging host 10.20.0.10
R1(config)# logging trap informational
R1(config)# logging source-interface loopback 0
```

Pinning the source interface stops messages arriving at the server with a different
address depending on the path, which breaks filtering and correlation.

### SNMP version 3

```text
R1(config)# snmp-server view READONLY iso included
R1(config)# snmp-server group MONITOR v3 priv read READONLY
R1(config)# snmp-server user nagios MONITOR v3 auth sha Snmp#Auth2026 priv aes 128 Snmp#Priv2026
```

If an old community string is still configured, remove it. It keeps working for as long
as it exists.

### Protecting image and configuration

```text
R1(config)# secure boot-image
R1(config)# secure boot-config
R1# show secure bootset
```

### Checking

```text
R1# show ip ospf interface gigabitEthernet 0/0
R1# show ip ospf neighbor
R1# show ntp associations detail
R1# show ntp status
R1# show logging
R1# show snmp user
R1# show secure bootset
R1# show ip interface gigabitEthernet 0/1 | include proxy|redirect|unreachable
```

`show ntp status` has to report the clock as synchronised, with a stratum. An
unauthenticated server shows up in `show ntp associations detail` without the
authenticated flag, and the client keeps using its time if you forget `ntp trusted-key`.

## Course labs

Anyone with a Networking Academy account will find this subject in labs 6.2.7, 6.3.6,
6.3.7, 6.6.4, 6.7.11 and 6.7.12 of module 6.

The next chapter moves credentials out of the device and into a server, which is what
makes a network of more than three routers manageable.
