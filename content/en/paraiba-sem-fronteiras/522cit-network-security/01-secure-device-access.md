---
title: "Secure device access"
description: "Why the management plane is the first target, what a stored password is worth, and how privilege should be distributed."
date: 2026-08-19
chapter: 1
translationKey: "psf-522-01"
weight: 10
tags: ["cisco", "ios", "ssh", "hardening"]
---

A router leaves the factory ready to be administered by whoever gets there first. It
accepts Telnet, keeps its password as readable text inside the configuration file, and
grants full privilege to anyone who knows a single word. In a university lab that word
tends to survive three semesters.

This chapter covers what has to change about that initial state, and the reason behind
each change. A firewall, an IPS and a VPN protect nothing on a device that accepts
unencrypted remote login, which is why the track starts here.

## Three planes, and where it hurts most

A network device runs three planes. The data plane forwards user packets. The control
plane exchanges routes, maintains the topology, and decides where traffic goes. The
management plane is where you configure the other two.

The gap in impact between them is not a matter of degree. Compromise the data plane and
an attacker reads the traffic crossing that point. Compromise the management plane and
the same attacker rewrites the access lists, switches off event logging, builds an
outbound tunnel, and erases the trace of the visit. You defend the first two planes
using the third, which makes the third the highest-return target on the box.

## What a stored password is worth

IOS keeps passwords in several formats, and the choice between them decides what happens
when the configuration file leaks. And it leaks: into backups, attached to support
tickets, committed to a git repository, emailed to a vendor.

Type 0 writes plain text. Type 7 applies a Vigenère cipher with a published key that any
web page reverses in a second. That format exists to frustrate someone reading over your
shoulder, and it achieves nothing beyond that. Type 5 uses MD5, fit for purpose in 1995
and broken for this use since the 2000s, because a GPU tests billions of candidates per
second. Types 8 and 9, PBKDF2 and scrypt, were designed to run slowly on purpose, and
scrypt also demands memory, which raises the cost of attacking it on dedicated hardware.

A common trap follows from this. The command that "encrypts" the passwords in the file
applies type 7, which is reversible. It changes how the configuration looks without
changing how safe it is. Anyone who sees an encrypted-looking file and concludes it is
safe to share has read the command backwards.

## A shared credential costs you accountability

A line password authenticates the device, not the person. When three students use the
same credential, the log stores three identical sessions, and the question of who wiped
the configuration has no answer.

Named accounts exist to separate authentication from accountability. The first decides
whether you get in. The second decides whether, months later, anyone can reconstruct
what happened. An audit trail without individual identity is a list of events with no
subject.

## Why Telnet left the stage

Telnet carries the username, the password and the rest of the session as clear text. All
an attacker needs is a vantage point along the path: a mirrored switch port, ARP
poisoning on the segment, a compromised intermediate router. Nothing has to be broken,
only read.

SSH solves two problems at once. It encrypts the session, which defeats passive capture,
and it authenticates the server through a host key, which makes impersonating the router
harder. The first connection still asks you to trust what you see, so comparing the key
fingerprint over a separate channel is worth the minute it takes.

Two choices inside SSH matter. Version 1 has integrity flaws and should not be accepted,
so only version 2 counts. And the RSA key needs 2048 bits at minimum, because 1024 fell
out of the range anyone considers safe.

## Where management should live

Encryption protects the content of a session. It does nothing to reduce who can knock on
the door.

The strongest arrangement is an out-of-band management network, physically separate from
user traffic and reachable only from where administrators sit. When budget or topology
rule that out, the fallback is filtering by source address on the remote access lines,
accepting connections from the administrative range alone.

The two measures solve different problems and stack. Encryption stops anyone reading
your session. Source restriction stops most of them from getting far enough to try.

## The cost of guessing

A dictionary attack depends on cheap attempts. Making them expensive changes the
arithmetic: the device closes login for a period after a number of failures inside a
time window, and records every failure.

This control carries a side effect that turns up both in exams and on call. If the
lockout applies to everyone, an attacker takes down your administrative access on
purpose by failing logins until the device shuts the door. So the lockout needs an
exception for the management range, or you have traded a break-in risk for a denial of
service against yourself.

## The banner is a legal instrument

An access warning is not decoration. In a prosecution for unauthorised access, the
defence argues that the system stood open to the public and that nobody could have
known. Explicit text about restriction, logging and legal consequence defeats that
argument. The word "welcome" supports it, which is why it has no place on a device
banner.

## Privilege that matches the role

IOS offers 16 privilege levels. Level 1 opens user mode, level 15 opens everything, and
the 14 in between sit empty until somebody fills them. The workshop assistant who only
needs to read interface status has no use for level 15.

That numbered model has a structural limit: it inherits upward. Whoever holds level 5
carries everything level 1 carries, and you cannot describe two parallel roles whose
command sets overlap without one containing the other. Role-based views exist for that.
Each view lists the commands its role runs, with no hierarchy between views, and a
superview groups them when one technician holds several roles.

The principle behind both mechanisms is least privilege: each account gets what the job
requires and nothing more, because every extra permission is a permission the attacker
inherits when that account falls.

## Verification is part of the control

Configuring without checking produces a feeling of security, which does more damage than
acknowledged insecurity, because it stops the investigation.

Four questions close the chapter. Which protocols do the remote access lines actually
accept, and is Telnet still among them? How large is the key in use? Is attempt blocking
armed, and how many failures has the device counted so far? Which accounts exist, and
did any of them end up with more privilege than the role calls for?

## Practice

The configuration below applies the whole chapter to one router. Build a router, a
switch and two PCs in Packet Tracer, one PC inside the management range 10.20.0.0/24
and the other outside it.

### Passwords and accounts

Set the minimum length before creating any credential, since IOS accepts the ones that
already exist and only enforces the rule on new ones:

```text
Router(config)# security passwords min-length 10
Router(config)# enable algorithm-type scrypt secret Ufcg#2026Redes
Router(config)# no enable password
Router(config)# service password-encryption
```

That `no enable password` matters. With both lines present, IOS authenticates against
`enable secret` and leaves the other password readable in `running-config`.

Create named accounts instead of relying on a line password:

```text
Router(config)# username julio privilege 15 algorithm-type scrypt secret Exchange#25
Router(config)# username operator privilege 5 algorithm-type scrypt secret Workshop#25
```

### SSH instead of Telnet

The RSA key derives from the device name and the domain, so set both before generating
it. Getting the order wrong produces a key with the wrong name and SSH refuses to start:

```text
Router(config)# hostname R1
R1(config)# ip domain-name redes.ufcg.br
R1(config)# crypto key generate rsa general-keys modulus 2048
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 2
```

On IOS-XE 16 and later the command lost its hyphen and became `ip domain name`.

Now close the remote access lines:

```text
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
```

Without `login local`, the line keeps asking for the line password and your named
accounts do nothing. Leaving `transport input all` keeps Telnet alive next to SSH and
undoes the work.

The auxiliary port deserves the same treatment, since almost nobody uses it and
therefore nobody watches it:

```text
R1(config)# line aux 0
R1(config-line)# no exec
R1(config-line)# transport input none
```

### Restrict the source, contain the attempt

```text
R1(config)# ip access-list standard MGMT
R1(config-std-nacl)# permit 10.20.0.0 0.0.0.255
R1(config-std-nacl)# exit
R1(config)# line vty 0 4
R1(config-line)# access-class MGMT in
R1(config-line)# exit
R1(config)# login block-for 120 attempts 3 within 60
R1(config)# login quiet-mode access-class MGMT
R1(config)# login on-failure log
R1(config)# login on-success log
```

`login quiet-mode access-class` is the exception the theory called for: during the
lockout the router still accepts connections from the management range, so you do not
lock yourself out of your own device.

### Banner

```text
R1(config)# banner motd ^
Authorised personnel only.
All activity on this system is logged and audited.
Unauthorised access is a criminal offence.
^
```

### Privilege

Numbered levels handle the simple case:

```text
R1(config)# privilege exec level 5 show ip interface brief
R1(config)# privilege exec level 5 show version
R1(config)# privilege exec level 5 ping
```

Parallel roles need views, and views need AAA switched on:

```text
R1(config)# aaa new-model
R1# enable view
R1(config)# parser view MONITOR
R1(config-view)# secret Workshop#Ver25
R1(config-view)# commands exec include show ip interface brief
R1(config-view)# commands exec include show ip route
R1(config-view)# commands exec include ping
```

### Check

```text
R1# show ip ssh
R1# show ssh
R1# show login
R1# show users
R1# show parser view all
R1# show running-config | include username|enable
```

`show ip ssh` confirms version 2 and the key size. `show login` reports whether blocking
is armed and how many failures the router counted.

Close by testing from both sides: open SSH from the PC inside the management range and
confirm access, try from the PC outside it and fail, get the password wrong four times
and watch the counter climb in `show login`.

One warning before leaving configuration mode: `login local` with no user created locks
you out, and recovery means a reboot and a password break.

## Course labs

This guide does not reproduce the labs. Anyone with a Networking Academy account and
Packet Tracer installed will find this chapter's subject in labs 4.4.7, 4.4.8 and 4.4.9
of module 4, and in 5.2.5 of module 5.

The next chapter moves to the rest of the device: services nobody uses that stay switched
on, routing protocols that accept any neighbour, and logs that reach nowhere.
