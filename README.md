# code-replay

Code Replay aims to build an evidence-driven agent for reconstructing the design history and decision process of GitHub projects.

It is not intended to be a generic repository summarizer or another coding agent. Its purpose is to help users continuously investigate why a project became what it is: why modules exist, why abstractions were introduced, how APIs evolved, which alternatives were rejected, and which explanations are supported by real project evidence.

## Mission

Build a conversational research agent that can work with users over time to understand a GitHub project through its code, Git history, pull requests, issues, reviews, documentation, and user corrections.

The system should maintain a project-level understanding that is:

- evidence-backed
- traceable
- revisable
- logically consistent
- usable through ongoing dialogue

The core output is not a one-time report. It is a living, queryable, and correctable understanding of a project's design evolution.

## What The Agent Should Answer

The agent should help users ask and answer questions such as:

- What is the current architecture of this project?
- Why does this module exist?
- Why was this abstraction introduced?
- How did this API, dependency, or module boundary evolve?
- Which design claims are directly supported by evidence?
- Which claims are only inferred?
- Has any previous understanding been contradicted by newer evidence?
- Why did the project choose one approach instead of another?

## Core Principles

### Evidence First

Every design explanation should be traceable to concrete evidence such as:

- commits
- diffs
- pull requests
- issues
- review comments
- documentation
- release notes
- code structure
- user corrections

### Separate Facts From Inference

The system must distinguish between:

- facts directly stated by evidence
- reasonable inferences
- speculation
- uncertainty
- contradiction

Confidence should be explicit, not hidden inside fluent prose.

### Long-Term Project Memory

The system should maintain memory around a project, not just around a single chat session. It should remember:

- confirmed design conclusions
- unresolved questions
- user corrections
- disputed claims
- stale explanations
- current research focus

### Belief Maintenance

Project understanding should be represented as a revisable belief system.

When a claim is contradicted or corrected, dependent claims and affected summaries should be marked stale, revised, downgraded, or rejected.

The agent should not simply append new explanations on top of old ones. It should maintain consistency.

### Layered And Multi-Dimensional Retrieval

Project information should be organized from raw evidence to high-level beliefs:

- raw evidence
- code entities
- local context
- change events
- design claims
- belief snapshots

It should also be indexed across dimensions such as time, causality, design intent, evidence strength, stability, impact scope, social collaboration, and user understanding.

### Query-Aware Context Assembly

Each user question should activate only the relevant parts of the knowledge system.

For example:

- a function-level question may need code graph and tests
- a design-motivation question may need Git history, PRs, issues, and claims
- a contradiction question may need claim graph, belief snapshots, and user corrections

The goal is minimal sufficient context, not maximum context.

### User Understanding Layer

The agent should not only understand the project. It should also maintain a
transparent, user-controlled understanding of the user's knowledge structure,
familiar domains, goals, corrections, preferred analogies, and explanation
style.

Effective answers come from aligning project knowledge with the user's cognitive
structure.

### Harness As Core Technology

The LLM is the reasoning engine, but the harness is what makes the system reliable.

Important harness components include:

- tool orchestration
- context management
- evidence retrieval
- claim extraction
- belief revision
- verification
- trace logging
- permission boundaries
- evaluation

### Domain Evaluation

The project should be evaluated not only as a coding agent, but as a design history reconstruction system.

Key evaluation targets include:

- evidence recall
- citation accuracy
- claim correctness
- unsupported claim rate
- contradiction detection
- stale propagation
- revision correctness
- long-term consistency

## Project Direction

The intended system can be summarized as:

```text
question
-> query routing
-> targeted retrieval
-> evidence tracing
-> claim extraction
-> consistency checking
-> belief revision
-> evidence-grounded answer
-> memory update
```

The broader architectural direction is:

```text
Layered Retrieval
+ Multi-dimensional Abstraction
+ Knowledge Compilation
+ User Understanding Layer
+ ClaimGraph / BeliefGraph
+ Belief Revision
+ Conversational Agent Harness
```

## Documents

More detailed design notes are available in `docs/`:

- `docs/agent-harness-technology-notes.md`
- `docs/agent-evaluation-strategy.md`
- `docs/hierarchical-knowledge-and-retrieval-architecture.md`
- `docs/user-understanding-layer.md`
- `docs/githru-reference-analysis.md`
- `docs/githru.pdf`
- `docs/reference-projects-and-products.md`
