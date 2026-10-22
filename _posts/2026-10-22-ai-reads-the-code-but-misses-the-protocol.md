---
title: "AI reads the code but misses the protocol"
category: writeup
tags: [ai, auditing]
---

I've spent a lot of time using AI tools to hunt for smart-contract vulnerabilities, and building frameworks to make that work better. One pattern keeps coming back. A model can read a large codebase, explain each function and spot familiar bug classes, yet still miss a vulnerability that depends on how several parts of the protocol interact.

A better prompt or a bigger context window doesn't fix that. Finding a bug takes more than understanding code. You have to question assumptions, connect execution paths that are far apart, notice unusual state transitions, and keep digging after the first explanation sounds reasonable.

## Where it goes wrong

Four problems show up again and again in my experiments.

### It matches patterns, and real bugs don't follow patterns

Models are good at recognizing vulnerability classes they've seen before. But unusual bugs usually come from interactions specific to one protocol, and those don't look like any textbook example.

### Correct in isolation, broken in combination

A function can look fine on its own and still break an invariant once it's combined with another contract, an external callback or a later state change. Without an explicit map of those relationships, the model misses the connection.

### A plausible explanation ends the search

Comments, docs and tests describe what the code is supposed to do. They don't prove the code does it. A convincing explanation of why an attack can't work is not evidence that it can't.

### It stops before verifying

A model may describe a possible vulnerability without ever building the state needed to trigger it. The opposite happens too: a passing test gets treated as proof of safety, even when it only covers one narrow scenario.

These come from my own work. How often each one happens depends on the model, the task, the tools available and how the investigation is set up.

## What I'm trying instead

Rather than handing an agent a repository and asking it to find bugs, I build a structured picture of the protocol first: the code structure, how contracts and functions relate, who the actors are, where assets flow, and which security properties must hold across state changes. The investigation can then target specific attack paths, with only the relevant code and dependencies in context. I described the design in [Structure the protocol before asking AI to find bugs](/2026/10/structure-the-protocol-before-asking-ai/).

Every suspected issue then needs a concrete case: the preconditions, the execution path, the property it breaks and the impact. Tests and proof-of-concept exploits decide whether it survives contact with the real implementation.

The aim isn't to make a model imitate human intuition. It's to turn part of the work behind good intuition into an explicit process that can be repeated.

## What would prove it works

A more structured workflow isn't automatically a better auditor. It has to find real vulnerabilities that a simpler workflow misses, without a big jump in false positives or cost.

Testing that means running both approaches on known vulnerabilities and on cases neither has seen, recording what each one misses, and checking that every claimed attack can actually be reproduced.

I'm not claiming this gives AI human-level auditing ability. That's the question I want to answer. My working hypothesis: **AI-assisted auditing gets better when protocol understanding, investigation depth and evidence checks are built into the workflow, instead of being left to the model.**
