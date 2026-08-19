---
title: "Cyber security in management"
description: "Cyber risk seen by whoever administers: policy, people, suppliers, and the response to an incident."
date: 2026-08-19
chapter: 4
translationKey: "psf-424-04"
weight: 40
tags: ["cybersecurity", "management", "gdpr", "incident"]
---

The network security guide in this same educational product covers how to configure the
defence. This chapter covers the decisions sitting before and after it, taken by people
who will never open a terminal.

## The shift in perspective

For whoever administers, security is not a technical problem with a technical solution. It
is a category of business risk, and it competes for budget against refurbishing a building
and against hiring.

That changes the question. Instead of "is this control the best available", the question
becomes "what expected loss does this control avoid, and does it cost less than that". The
answer is rarely obvious, which is why the risk management chapter exists.

It changes the vocabulary too. A board does not decide about firewalls; it decides how much
downtime the organisation tolerates and how much it will spend to reduce that time.

## Where failure actually enters

The popular image of the sophisticated technical attack distorts how resource gets
allocated. Most incidents start with something mundane: somebody clicked, somebody reused a
password, somebody left a service exposed without knowing, somebody kept access after
leaving.

That repositions the security policy as a management instrument rather than a compliance
document. A useful policy answers who has access to what and for how long, what happens
when someone joins and when they leave, how information is classified by sensitivity, what
is acceptable on a personal device, and who tells whom when something goes wrong.

A policy nobody reads fails the same way an emergency plan nobody tests fails.

## People

Security training has a deserved bad reputation, because most of it is an annual video
followed by a quiz.

What works is specific, short, and tied to the job of whoever watches. The finance team
needs to recognise payment fraud, not learn what asymmetric cryptography is. And a simulated
exercise, when used to measure and adjust rather than to punish, teaches more than the
video.

One point of culture decides the outcome: if reporting your own mistake is expensive for
whoever reports it, nobody reports, and the organisation learns about the incident weeks
later from a customer.

## Suppliers

A large share of an organisation's risk surface lives outside it, in contracted services.
The contract is where management acts.

It is worth knowing what to ask before signing: where the data is stored, who on the
supplier's side has access, what happens during an incident and how quickly they notify you,
how the data comes back at the end of the contract, and what happens if the supplier is
acquired or shuts down.

Responsibility to the customer and to the law stays with whoever contracted, which repeats
the point about transfer from the risk chapter.

## When it happens

Incident response is a plan, and the plan has to exist beforehand.

It defines who leads, what happens in the first hours, when and how to tell customers and
the regulator, and who speaks to the press. Data protection law imposes a notification
deadline, and discovering that during the incident is expensive.

Then comes the part most people skip: the analysis of what happened, done to fix the cause
rather than to find a culprit. An organisation treating the post-incident review as a
disciplinary hearing learns once and never again.

## In practice

Pick a small organisation you know, a clinic, a shop, or a university lab.

List its five most important information assets and, for each, what would happen if it
leaked, if it were altered without authorisation, or if it were unavailable for a week. The
three questions map onto confidentiality, integrity and availability.

Then map who has access to each, and check whether anyone holds access their job does not
require. That check alone usually yields findings.

Next, write the first three actions of the first hour of a ransomware incident at that
organisation, with a named owner.

Finally, list the external services it depends on and what happens if one of them is
unavailable for three days.

## Going further

The module covers cyber security from the resource management viewpoint, which complements
the technical guide on this same site. Reading both in sequence shows the same threat
described in two vocabularies.

The next chapter widens the focus to technology's effect on the whole organisation.
