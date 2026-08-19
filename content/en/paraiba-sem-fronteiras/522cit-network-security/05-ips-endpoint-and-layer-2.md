---
title: "IPS, endpoint and layer 2"
description: "Intrusion detection, host defence, and the attacks that happen on the bottom floor of the stack."
date: 2026-08-19
chapter: 5
translationKey: "psf-522-05"
weight: 50
tags: ["ips", "ids", "endpoint", "layer2", "switch"]
---

A firewall decides on address, port and connection state. That lets through everything
malicious and well formed at the same time: an HTTP request carrying SQL injection
arrives on port 443 inside a legitimate session. This chapter covers the three layers of
defence that continue after filtering.

## Detecting and preventing are different positions

An intrusion detection system receives a copy of the traffic, analyses it off the path,
and raises an alert. It delays nothing and blocks nothing, so a successful attack
produces an alert about damage that already happened.

A prevention system sits in the path. Every packet crosses the analysis before moving
on, which allows dropping the malicious session before it reaches its target. The price
shows up in two places: the latency the analysis adds, and the false positive, which in
inline mode drops legitimate traffic instead of raising an alert somebody ignores.

Choosing between them is a choice about the cost of being wrong. On a lab network a
false positive costs a support ticket. On a payment system it costs revenue per minute.

## The four possible outcomes

Every detection decision falls into one of four boxes, and exams ask for the names.

A true positive alerts on a real attack. A true negative stays quiet during legitimate
traffic. A false positive alerts on normal traffic. A false negative stays quiet during
an attack.

The two errors do not cost the same. A false negative is the intrusion nobody saw. A
false positive is noise, and enough noise produces something worse than the error
itself: the team stops reading alerts. Tuning an IPS consists almost entirely of
reducing false positives without creating false negatives, and that tuning never ends.

## How the system decides

Signature detection compares traffic against known patterns. It is precise and blind to
anything without a signature yet, which includes every new attack.

Anomaly detection learns the network's normal behaviour and alerts on deviation. It
catches what has no signature and produces false positives whenever normal changes,
which happens every time someone deploys a new service.

Policy detection blocks what the organisation declared forbidden, without judging
whether it is an attack. And a honeypot draws the attacker toward a fake target, useful
for studying technique and for diverting attention.

When the system acts, the options run from logging the event to dropping the packet,
resetting the connection, or blocking the source address for a period.

## Where defence lives

A network sensor sees traffic from many hosts and sees nothing of what happens inside
them. It also goes blind against encrypted traffic, which today is most of it.

A host agent sees the opposite: system calls, file changes, processes starting, and
content after decryption. It costs installation and maintenance on every machine, and an
attacker who gains privilege on the host switches the agent off.

The two views complement each other, and endpoint defence today combines antimalware,
application control, disk encryption, and posture checking before granting network
access. That last one closes the circle with the previous chapter: an unpatched machine
lands on a restricted network until it complies.

## The bottom floor

Everything the earlier chapters built assumes layer 2 works. It is the foundation, and
whoever compromises it never has to fight what sits above.

MAC table flooding fills the switch with fake addresses until it stops learning and
starts flooding every frame out of every port, which turns the switch into a hub and
hands the traffic to anyone listening.

VLAN hopping crosses the separation you created. In one variant the attacker negotiates
a trunk link with the switch and starts receiving every VLAN. In the other they insert
two tags into the frame, the first switch strips one and forwards, and the second
delivers the frame into the target VLAN.

Two DHCP attacks combine. The first exhausts the address pool by requesting leases with
forged source addresses. The second stands up a rogue server that answers before the
real one and hands out itself as the gateway, which puts the attacker in the middle of
every conversation.

ARP poisoning exploits a protocol that believes any reply. The attacker claims to be the
gateway to the victim and the victim to the gateway, and sees both directions.

And a spanning tree root attack advertises a better priority, wins the election, and
redraws the topology to run through the attacker's device.

## The countermeasure set

Each attack above has a specific answer, and they depend on one another.

Port security limits how many addresses a port learns, which handles table flooding.
DHCP snooping classifies ports as trusted or untrusted, drops server replies arriving on
untrusted ports, and builds a table binding IP address, hardware address and port.
Dynamic ARP inspection uses that table to drop ARP replies that fail to match it, which
is why it depends on snooping being enabled. BPDU guard shuts down any access port that
receives a spanning tree message, since no computer should send one.

Three configuration habits close the rest. Disable automatic trunk negotiation on access
ports so nobody negotiates a trunk with you. Change the native VLAN to one that carries
no user data, which breaks double-tagging. And leave unused ports shut down and assigned
to a VLAN that goes nowhere.

## Practice

A switch with PCs on `f0/1` to `f0/5`, the legitimate DHCP server on `f0/24`, and the
router on `g0/1`.

### Port security

```text
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport port-security
SW1(config-if-range)# switchport port-security maximum 2
SW1(config-if-range)# switchport port-security mac-address sticky
SW1(config-if-range)# switchport port-security violation restrict
SW1(config-if-range)# switchport port-security aging time 60
```

The violation action has three settings. Protect drops silently, restrict drops and
logs, and shutdown disables the port until someone intervenes. Shutdown is the safest
and generates the most tickets, because a user moving desks takes down their own port.

### DHCP snooping

```text
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10
SW1(config)# no ip dhcp snooping information option
SW1(config)# interface fastEthernet 0/24
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# ip dhcp snooping limit rate 6
```

### Dynamic ARP inspection

```text
SW1(config)# ip arp inspection vlan 10
SW1(config)# interface fastEthernet 0/24
SW1(config-if)# ip arp inspection trust
SW1(config-if)# exit
SW1(config)# ip arp inspection validate src-mac dst-mac ip
```

### Spanning tree and trunks

```text
SW1(config)# spanning-tree portfast default
SW1(config)# spanning-tree portfast bpduguard default
SW1(config)# interface gigabitEthernet 0/1
SW1(config-if)# spanning-tree guard root
SW1(config-if)# exit
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# switchport nonegotiate
SW1(config-if-range)# exit
SW1(config)# interface gigabitEthernet 0/2
SW1(config-if)# switchport trunk native vlan 999
```

### Idle ports

```text
SW1(config)# interface range fastEthernet 0/6 - 23
SW1(config-if-range)# switchport access vlan 999
SW1(config-if-range)# shutdown
```

### Checking

```text
SW1# show port-security
SW1# show port-security address
SW1# show ip dhcp snooping
SW1# show ip dhcp snooping binding
SW1# show ip arp inspection
SW1# show ip arp inspection statistics
SW1# show spanning-tree inconsistentports
SW1# show interfaces status err-disabled
```

The snooping binding table is the central artefact. If it sits empty, ARP inspection
drops legitimate traffic, and the symptom is a network that stops right after you
believed you had secured it.

## Course labs

Modules 11 to 14 ship no Packet Tracer lab. The practice above builds with one switch
and three PCs, and testing the offensive side needs a machine with traffic generation
tools, which Packet Tracer does not simulate.

The next chapter changes subject and moves to the mathematics underneath everything that
follows: cryptography.
