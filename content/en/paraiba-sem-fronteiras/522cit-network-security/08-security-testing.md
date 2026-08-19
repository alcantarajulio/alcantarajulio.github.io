---
title: "Network security testing"
description: "Verifying on the wire what you believe you configured, with written authorisation and a repeatable method."
date: 2026-08-19
chapter: 8
translationKey: "psf-522-08"
weight: 80
tags: ["pentest", "audit", "nmap", "wireshark"]
---

The previous seven chapters produced a configuration. This one deals with the gap
between the configuration you believe you applied and the behaviour the network actually
exhibits, which is where incidents live.

## Why verification is a separate discipline

You applied the access list to the wrong interface. You configured the zone-based
firewall and left one interface outside every zone. You disabled Telnet on three routers
and the fourth slipped through because the session dropped halfway. None of these
mistakes surfaces by reading the configuration carefully, because the reader is the
person who wrote it and reads what they meant to write.

Testing inverts the position. Instead of asking what the configuration says, it asks
what the network answers.

## Authorisation comes first

Scanning a network without authorisation is a crime. In Brazil, Law 12.737 criminalises
unauthorised access to computing devices; in the United Kingdom the Computer Misuse Act
covers the same ground with heavier sentences. Neither carves out an exception for
academic curiosity or good intentions.

Before any scan there is a document. It defines scope by address and by system, the time
window, which techniques are permitted, who the contacts are on both sides, and what
happens if the test takes down a service. Without it you are not running a security
test, you are committing a crime with good intentions.

In a workshop this has a practical consequence: build the lab on your own machines or in
an isolated virtual environment, and never point a tool at the institution's network.

## Assessment and penetration testing

The two terms get mixed up and describe different jobs.

A vulnerability assessment aims for breadth. It scans everything in scope, compares
against databases of known flaws, and produces a list of problems with severities. The
typical output is a long report with many items, and it does not confirm whether a flaw
is exploitable in your context.

A penetration test aims for depth. It starts from an objective, such as reaching the
database from the guest network, and pursues that path until it succeeds or runs out of
options. The output is short and concrete: this path works, and here is the proof.

The choice depends on the question. To learn what needs fixing, assessment. To learn
whether your defence holds, penetration test.

On how much information the tester receives, three arrangements exist. With none, the
test simulates an outside attacker and spends time on reconnaissance. With everything,
it finds more in less time and simulates the real world less well. The middle ground,
with an ordinary user credential and a network diagram, usually returns the most per
hour.

## The phases, and what the defender learns from them

A test moves through reconnaissance, scanning, gaining access, maintaining access, and
covering tracks. That sequence also describes what a real attacker does, which is why it
matters to whoever defends.

Every phase leaves a signal. Reconnaissance shows up as DNS queries and public
collection. Scanning shows up as sequential connections to closed ports. Gaining access
shows up as repeated authentication failures. Maintaining access shows up as a new
account or a scheduled task. Covering tracks shows up as a hole in the log.

Reading the phase list as a detection guide beats memorising it. If you configured the
remote logging from chapter 2, you have material to hunt each of those signals in. If
you did not, phase five works.

## The categories of test

The course material sorts by type, and the names are worth knowing. Network scanning
discovers what exists and what answers. Vulnerability scanning compares what answered
against known flaws. Password cracking measures credential strength. Integrity checking
compares files against a known reference. Log review searches what already happened. And
configuration integrity testing compares what is running against the approved baseline.

That last one deserves emphasis, because it closes the guide's loop. A baseline is the
configuration you declared correct. Comparing every device against it turns auditing
into an automatable routine, and turns drift into an alert.

## Practice

Build an isolated lab with the router from earlier chapters, a switch and a Linux
machine. Nothing here should touch anyone else's network.

### Discover what exists

```text
$ nmap -sn 10.20.0.0/24
```

A scan with no ports discovers live hosts. It is the first step and already reveals
equipment nobody knew was powered on.

### See what answers

```text
$ nmap -sS -p- 10.20.0.1
$ nmap -sV -O 10.20.0.1
```

The result is your chapter 2 checklist. If port 23 comes back open, Telnet survived
somewhere. If port 80 comes back, the built-in HTTP server is still running. If a
service you do not recognise appears, you have just found the reason this chapter exists.

### Confirm what the filtering does

```text
$ nmap -sS -p 22,23,80,443 10.30.0.10
$ nmap -Pn --reason -p 80 10.30.0.10
```

`--reason` shows why nmap classified a port the way it did, and the difference between a
refusal and no answer at all distinguishes a closed port from a filtered one. That
distinction is the real test of your access list.

### Demonstrate chapter 1 with a capture

Open Wireshark, run a Telnet session against an unconfigured test device, and follow the
TCP stream. The password appears character by character. Repeat with SSH against the
configured device and show the same stream as unreadable.

This is the most effective demonstration in the entire workshop, and it costs two
minutes.

### Measure credential strength

Take a type 5 digest and a type 9 digest from your own lab and submit both to the same
dictionary cracking attempt. The difference in time between them turns chapter 1's
argument into a number, and that is the kind of evidence that convinces whoever controls
a budget.

### Audit the configuration

```text
R1# show running-config | include ^line|transport|login|password|username
R1# show ip interface brief
R1# show archive config differences system:running-config nvram:startup-config
R1# show ip access-lists
R1# show crypto session
```

Compare the output against your baseline. A divergence between the running and the saved
configuration usually indicates an undocumented change, and that is among the first
things to look for after an incident.

## Course labs

Module 22 ships no Packet Tracer lab, and much of the practice above needs real tools on
a Linux machine, which the simulator does not replace. The module's practical
assessment, the Skills Assessment, covers the configuration from the earlier chapters
together.

This is the last chapter of the guide. The eight together cover the full cycle: protect
the device, protect the control plane, control who administers it, filter the traffic,
detect what got through, understand the cryptography, apply it in a tunnel, and return
to the start to verify that any of it works.
