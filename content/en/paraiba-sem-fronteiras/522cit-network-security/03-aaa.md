---
title: "AAA: authentication, authorization and accounting"
description: "Moving credentials off the device, choosing between RADIUS and TACACS+, and not locking yourself out along the way."
date: 2026-08-19
chapter: 3
translationKey: "psf-522-03"
weight: 30
tags: ["aaa", "radius", "tacacs", "cisco"]
---

Local accounts work fine across three routers. Across thirty they become a
synchronisation problem: someone joins the team and you configure thirty devices,
someone leaves and you miss one, and six months later an audit finds that last
semester's intern still logs into the edge router.

## Three different questions

The acronym bundles three functions that solve different problems and get confused in
exams.

Authentication answers who you are. Authorization answers what you may do once you are
in. Accounting answers what you did, and it is the one that holds up an audit.

A system can authenticate without authorizing at any granularity, which is what happens
when every administrator lands on level 15. It can also authenticate and authorize
without recording anything, and then you learn what happened by asking people.

## Local and centralised

Local AAA keeps the user database on the device itself. It already beats a line
password, because it carries individual identity, and it still suffers the scale
problem.

Centralised AAA puts the database on a server. Revocation happens in one place, password
policy applies uniformly, and access records for the whole network land in one
repository. The cost is a new dependency: if the server goes down and you planned no
alternative, nobody gets into anything.

That alternative is called a method list, and it is the part that catches students in
the lab. You declare the order of attempts: the server first, the local database second.
The device falls through to the second method only when the first fails to answer. A
server that answers with a denial does not trigger the fallback, and the difference
between "did not answer" and "answered no" is the difference between getting in and
staying out.

## RADIUS and TACACS+

Both protocols do AAA and serve different purposes.

RADIUS runs over UDP, encrypts only the password field and leaves the rest of the packet
readable, including the username. It combines authentication and authorization into one
exchange, which reduces granularity. It is an open standard, implemented everywhere, and
it dominates end-user access: wireless, 802.1X, VPN.

TACACS+ runs over TCP, encrypts the entire packet body, and splits the three functions
into independent exchanges. That separation allows per-command authorization: the server
decides, for each line typed, whether that administrator may run it. The protocol comes
from Cisco, and its natural use is network device administration.

The practical answer is usually both. TACACS+ for the people who administer the
equipment, RADIUS for the people who merely use the network.

## Accounting is what remains afterwards

AAA records answer questions that device syslog answers poorly. Who opened a session, at
what time, from where, for how long, and which commands they ran. With TACACS+ and
per-command authorization, the record holds the exact line.

That carries a side effect worth saying out loud in a workshop: full accounting records
people's behaviour, and there is a conversation about proportion and transparency that
is not technical. Recording administrator commands on critical equipment is common and
defensible. Recording everything about everyone without telling them is something else.

## Practice

The scenario adds an AAA server at 10.20.0.20. Packet Tracer ships a server with RADIUS
and TACACS+ built in, which is enough for the lab.

### Build the fallback before anything else

This order prevents the disaster. Create the local account first, then enable AAA:

```text
R1(config)# username julio privilege 15 algorithm-type scrypt secret Exchange#25
R1(config)# aaa new-model
```

### Point at the servers

```text
R1(config)# tacacs server AAA-TAC
R1(config-server-tacacs)# address ipv4 10.20.0.20
R1(config-server-tacacs)# key Tac#Redes2026
R1(config-server-tacacs)# exit
R1(config)# aaa group server tacacs+ GROUP-TAC
R1(config-sg-tacacs+)# server name AAA-TAC
```

For RADIUS, the equivalent:

```text
R1(config)# radius server AAA-RAD
R1(config-radius-server)# address ipv4 10.20.0.20 auth-port 1812 acct-port 1813
R1(config-radius-server)# key Rad#Redes2026
```

### Method lists

The default list applies to anything without a list of its own. The named list for the
console is your insurance against losing the server:

```text
R1(config)# aaa authentication login default group GROUP-TAC local
R1(config)# aaa authentication login CONSOLE local
R1(config)# aaa authentication enable default group GROUP-TAC enable
```

Apply the console list to the matching line:

```text
R1(config)# line console 0
R1(config-line)# login authentication CONSOLE
```

### Authorization and accounting

```text
R1(config)# aaa authorization exec default group GROUP-TAC local
R1(config)# aaa authorization commands 15 default group GROUP-TAC local
R1(config)# aaa accounting exec default start-stop group GROUP-TAC
R1(config)# aaa accounting commands 15 default start-stop group GROUP-TAC
```

The trailing `local` on the authorization lines matters as much as it does on
authentication. Without it, an unreachable server means an authenticated administrator
who cannot run anything.

### Check before closing the session

Never close your current session without testing in another one. Open a second terminal
and verify:

```text
R1# test aaa group GROUP-TAC julio Exchange#25 legacy
R1# show aaa servers
R1# show aaa sessions
R1# debug aaa authentication
```

`test aaa` validates communication with the server without depending on a fresh login.
`debug aaa authentication` shows which method the list tried and in what order, which is
the information missing when a login fails with no clear message.

### The classic mistake

Enabling `aaa new-model` with no local user and no reachable server locks the device.
`aaa new-model` changes default line behaviour immediately, and the line password that
worked a second ago stops working. Recovery means physical console access and a password
break at boot.

## Course labs

The subject appears in labs 7.2.5 and 7.2.6 of module 7.

The next chapter leaves control over who administers the device and moves to control
over the traffic crossing the network.
