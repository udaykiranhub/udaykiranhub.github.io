---
title: "Structure the protocol before asking AI to find bugs"
category: writeup
tags: [ai, auditing, tooling]
---

AI models are good at reading a function and pointing out something suspicious. They can explain an attack scenario and suggest where a bug might be. But the bugs that matter in real protocols rarely live inside one function. They come from how contracts, state changes, accounting rules and validation checks interact.

To find those, you need to know how the pieces relate and which conditions must hold across all of them. That is exactly what models lose track of. They miss cross-contract dependencies, chase the same hypothesis twice, or get so much irrelevant code that the important part gets buried.

I'm building an AI-assisted smart-contract security engine around one idea: **model the protocol first, then ask the AI to look for bugs.** This post explains the design. It's a work in progress, and nothing here is a benchmark result yet.

## Where the usual workflow breaks

The common approach is simple: give the model some source files, name a vulnerability class, and ask it to investigate. That works for local bugs. It struggles with bugs that span several components.

Take a lending protocol with deposits, borrowing, position transformations and liquidations. The borrow function may validate everything correctly, while another operation changes an existing position through a different path. Whether that second path still respects the borrowing limits depends on both operations, the accounting underneath them, and the validation logic.

Show the model only the borrow function and it misses the interaction. Show it the whole repository and the interaction is buried in unrelated code.

Running more agents doesn't fix this on its own. They can walk the same paths, miss dependencies between hypotheses, or reach conclusions they can't prove. The problems feed each other: weak protocol understanding leads to bad context, bad context leads to weak reasoning, and weak validation lets plausible but wrong findings through.

## The pipeline

The engine turns a Solidity repository into structured representations before any agent looks for bugs:

```text
repository → parsing → code model → relationship graphs → protocol model
           → context engineering → security agents → validation → report
```

Each stage does one job. The work of understanding the code is done once and reused, instead of the model rebuilding the protocol from source on every run.

It also makes misses debuggable. If a bug is missed, I can check which stage failed: a relationship missing from the graph, a wrong invariant in the protocol model, context selection dropping the key evidence, or the agent reasoning badly.

## 1. Code model

The first stage reads the repository and records its contracts, functions, state variables, imports and project configuration. Tree-sitter is the main parser, and Solidity compiler AST output adds detail when it's available.

Nothing here tries to find bugs. It gives every later stage a clean representation of the code to work from.

## 2. Relationship graphs

Listing contracts and functions doesn't explain a protocol. The graph layer records how things connect: inheritance, function calls, storage reads and writes, and source dependencies. It uses NetworkX.

This is what links operations that look unrelated. A withdrawal might depend on a position value computed elsewhere, which depends on state that a third function modifies. An investigation has to follow that whole chain.

The graphs have limits. An edge between two functions doesn't mean the path is reachable, or that it breaks anything. The graphs are a map for later stages, not proof.

## 3. Protocol model

A graph shows how code connects. It doesn't say what those connections mean. That's the job of the `ProtocolModel`: actors, permissions, assets, value flows, accounting relationships, execution paths and invariants.

In a vault, for example, the code has deposit, withdraw, share-minting and accounting functions. The protocol model ties them to behavior: how a deposit changes assets and shares, how a withdrawal changes ownership claims, and which accounting relationships must always hold. A single function can look fine while a sequence of calls breaks an invariant, so this matters.

Every security agent receives the protocol model along with the code. The model also has to be honest about what it knows. Source code doesn't always state the intended economics or every external dependency, so the model keeps facts from the implementation separate from inferred relationships and open assumptions.

## 4. Context engineering

With the graphs and protocol model in place, the next question is what each agent actually gets to see.

For each investigation, the engine builds a `ContextBundle`. It selects the relevant functions, state variables, dependencies and protocol facts, ranks them, removes duplicates, compresses what isn't needed, and tracks token usage.

For a suspected bypass of a lending check, that means the position-transformation path, the debt accounting, the collateral valuation, the validation functions, and the conditions under which those checks run. Connected pieces, delivered together, instead of a pile of files.

There's a balance to strike. Too much context wastes tokens and hides details. Too little can cut out the one dependency that makes the bug possible. The aim is to keep everything needed to answer the security question and drop everything that isn't.

Because context building is its own component, I can measure how well it works without involving the LLM at all.

## 5. Running the investigation

Each agent declares the question it investigates, the context it needs, and the shape of its output. A shared framework handles execution, context budgets, logging and validation, so agents don't each carry their own orchestration code.

The point isn't that more agents find more bugs. It's that they all work from the same protocol model and produce hypotheses in a consistent, checkable form.

LLM providers sit behind one interface. Claude is the main model, with OpenAI as a fallback or a second reviewer.

## 6. Turning hypotheses into findings

Better context still doesn't stop a model from being wrong. An attack can be unreachable, a check can already block it, or the claimed impact may not follow from the code. So the engine judges findings by evidence, not by how convincing the explanation sounds.

Every candidate finding has to name the actor, the preconditions, the execution path, the constraint it breaks and the impact. It should also say what evidence would disprove it. If every reachable path calls the check the attack claims to skip, the finding is rejected. If the required starting state can't happen, there's no exploit.

The validation layer checks output structure, evidence, attack-path completeness and duplicates. Static analysis results can support a finding, and promising ones may need a targeted test or proof of concept. Generating PoCs automatically is out of scope for the first version.

An AI hypothesis is where an investigation starts. It isn't a confirmed vulnerability.

## 7. Traceability

I want to know not only what the engine reported, but how it got there.

The deterministic parts (parsing, graphs, context selection) should give the same output for the same inputs and tool versions. LLM output can change between runs. So each result records the context the agent received, the prompt hash and the agent version. When a result changes, I can tell whether the code changed, the graph changed, the context changed, or the model simply reasoned differently.

Failures are recorded the same way. If an agent crashes or times out, the report shows that gap. A missing analysis is never presented as a clean result.

## How I'll know if it works

All of this is a hypothesis until it's tested. The questions I need to answer:

- Are the code relationships represented correctly?
- Does the protocol model capture the invariants that matter?
- On known bugs, does context selection keep the evidence needed to find them?
- Can validation separate reachable attack paths from impossible ones, reject unsupported impact claims, and remove duplicates without throwing away real issues?

For context engineering, the fair test is a baseline: give a model the same source material directly, then compare it with the pipeline's context on the same cases, for both finding quality and token cost.

None of these are results yet. They're what the design has to prove. The question underneath it all is simple: does AI-assisted auditing get better when the protocol is modeled before the bug hunt starts? I'm building this engine to find out.
