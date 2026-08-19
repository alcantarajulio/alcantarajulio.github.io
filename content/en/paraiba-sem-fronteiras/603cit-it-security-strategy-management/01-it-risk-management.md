---
title: "IT risk management"
description: "Identifying, measuring and responding to risk without turning the register into a document nobody reads."
date: 2026-08-19
chapter: 1
translationKey: "psf-603-01"
weight: 10
tags: ["risk", "governance", "it"]
---

Every IT decision is a bet on the future. You pick a supplier without knowing whether it
exists in five years, migrate a system without knowing how long the migration takes, and
postpone a patch betting the flaw stays unexploited until the next window. Risk
management is the method that turns those implicit bets into declared decisions.

## A risk is not a problem

The distinction looks like formality and changes the vocabulary of the meeting. A problem
already happened and demands a response. A risk might happen, and demands a decision
beforehand.

A risk has three components: the event, the probability it occurs, and the impact if it
does. Describing risk without all three produces useless phrases like "security risk",
which nobody can prioritise or budget for.

The form that works names cause, event and consequence. Instead of "risk of server
failure", write "because the rack has a single power feed, the billing server could go
offline, which would stop invoicing for up to six hours". The second version already
contains its own answer.

## Measuring so you can compare

The probability and impact matrix is the most used instrument, and the most misused. You
place each risk on a grid, which allows comparing items with nothing in common, and
produces an order of attention.

The trap sits in the scale. When "high", "medium" and "low" stay undefined, everyone on
the team applies their own ruler, and the result aggregates incompatible opinions. The
fix anchors each level to a number: high impact means a loss above some figure, or an
outage above some number of hours. That anchoring turns a discussion of temperament into
a discussion of criteria.

Qualitative assessment suits fast triage and risks that resist quantification.
Quantitative assessment, multiplying probability by expected loss, suits cases with
history and decisions needing a financial justification. The two coexist: triage with the
first, go deeper with the second only at the top of the list.

## The four responses

After measuring, four paths exist, and choosing one is compulsory. Not choosing is
accepting by omission.

Avoiding means dropping the activity that creates the risk. Cancelling the integration
with the questionable supplier removes the risk and removes the benefit it would have
brought.

Transferring moves the financial consequence to another party, through insurance or a
contract with a service level clause. Note that transfer moves the money and leaves the
responsibility: when the system falls, your customers complain to you.

Mitigating reduces probability, impact, or both. Most security work lives here.

Accepting is the decision to live with it, and it is legitimate when the cost of the
control exceeds the expected loss. Acceptance has to be recorded with a name and a date,
because anonymous acceptance becomes negligence once the event happens.

## The risk register and why it dies

The register is the living list of risks with an owner, an assessment, a chosen response
and a deadline. It fails in two predictable ways.

The first is the register with no owner. A risk assigned to "the IT team" is assigned to
nobody, and the quarterly review finds the same item open two years running.

The second is the register nobody reviews. Risk has a shelf life: what was unlikely last
year may have become routine, and the control you deployed may have stopped working. A
register with no review cadence documents the past.

## What the disasters teach

The literature on IT risk management is built on expensive failures, and the cases repay
study because they repeat patterns.

The most common pattern is the known risk that never escalated. Someone on the team knew,
the warning climbed one level and stopped, and the decision was taken by people without
the information. The second is optimism in a chain: every layer of the hierarchy rounds
the estimate down, and the final deadline bears no relation to the work.

The third shows up in large public projects, and it is scope that grows while the budget
does not. It produces systems delivered years late with a fraction of the promised
function, and it is why this discipline exists as a subject of its own.

## In practice

Pick a system you know well, preferably one you operate yourself.

First, list five risks using the cause, event and consequence form. The format constraint
is the exercise, because writing that way forces you to know what you are talking about.

Second, define your scale before assessing. Write down what high, medium and low impact
mean as numbers in your context. Do the same for probability.

Third, place the five on the matrix and rank them.

Fourth, choose one of the four responses for each, with an owner and a deadline. If you
choose acceptance, write the justification: the cost of the control and the expected loss.

Fifth, and this is the step separating an exercise from a practice, set a review date and
keep it. A register reviewed twice beats a perfect register reviewed never.

## Going further

The module's material works with failure cases from public technology projects and with
risk analysis in high-consequence engineering. Both angles serve the same purpose: showing
that risk is rarely unknown, and that failure lives in the communication between those who
know and those who decide.

The next chapter leaves defence and turns to where technology creates advantage.
