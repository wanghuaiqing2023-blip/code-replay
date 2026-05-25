# User Understanding Layer

## 1. Core Idea

A project understanding agent should not only understand the project. It must
also understand the user it is explaining the project to.

For this project, the user is not a blank query endpoint. The user has an
existing knowledge structure, familiar domains, weak domains, goals, preferred
reasoning paths, accepted beliefs, rejected explanations, and useful analogies.

Without this layer, the agent may produce answers that are technically correct
but not cognitively effective.

The missing layer can be named:

```text
User Understanding Layer
```

or more directly:

```text
Me Layer
```

Its purpose is to let the agent align project explanations with the user's
knowledge structure and cognitive structure.

## 2. Why This Layer Is Necessary

The same project fact can require different explanations for different users.

For example:

```text
Why did the project introduce event sourcing?
```

Possible explanation paths:

- For a user familiar with databases: compare it to WAL or append-only logs.
- For a user familiar with Git: compare it to commit history.
- For a user familiar with accounting: compare it to a ledger.
- For a user familiar with compilers: compare it to transformation traces.

The answer's effectiveness depends not only on factual correctness, but also on
how well the explanation maps to what the user already knows.

Therefore, the system should optimize for:

```text
project truth + user alignment
```

not just:

```text
project truth
```

## 3. Two Compilation Targets

Earlier architecture discussions focused on compiling project knowledge:

```text
raw project evidence
-> Project Understanding Layer
```

This is necessary but incomplete. The system should also compile user
understanding:

```text
raw user interactions
+ preferences
+ corrections
+ known domains
+ reasoning patterns
-> User Understanding Layer
```

At answer time, the agent should combine both:

```text
Question
+ Project Understanding Layer
+ User Understanding Layer
-> Personalized Evidence-Grounded Answer
```

## 4. What The User Understanding Layer Should Contain

The User Understanding Layer should not be a vague profile. It should be a
structured, inspectable, editable model that helps the agent collaborate with
the user.

Possible dimensions:

```text
Knowledge Map
Which domains, technologies, concepts, and mental models the user already knows.

Unknown Map
Which domains or concepts the user is currently learning or finds weak.

Analogy Map
Which analogies have worked well for this user.

Rejected Analogy Map
Which analogies or explanation styles the user has rejected.

Goal Map
Why the user is studying this project and what outcome they care about.

Preference Map
Preferred answer length, language, rigor, structure, and level of detail.

Belief Map
Which claims the user has accepted, questioned, corrected, or rejected.

Interaction History
What the user has asked before and what research path they are following.

Cognitive Style
Whether the user prefers macro-to-micro, example-to-principle, principle-to-example,
or comparison-driven explanations.

Vocabulary Map
Terms, naming habits, preferred language, and recurring concepts used by the user.

Decision Criteria
What the user tends to value when judging technical decisions, such as simplicity,
performance, maintainability, correctness, cost, or extensibility.
```

## 5. Data Sources

The Me Layer can be built from multiple sources:

- explicit user input
- conversation history
- user corrections
- repeated follow-up patterns
- accepted or rejected explanations
- uploaded notes
- the user's own projects
- documents the user writes or references
- signals such as "this analogy is useful" or "do not explain it that way"

The system should treat user corrections as especially important. A correction
is not just conversation context. It should become durable evidence about how
the user understands the topic and how the system should answer in the future.

## 6. Answer Generation With User Alignment

A generic answer chain is:

```text
retrieve project context
-> generate answer
```

The intended chain should be:

```text
classify user question
-> retrieve project context
-> retrieve user context
-> choose explanation strategy
-> generate answer
-> observe feedback
-> update project memory and user memory
```

Example:

```text
Project Claim:
Queue was introduced to isolate third-party API rate limits.

User Understanding:
The user is familiar with operating systems and scheduling.

Answer Strategy:
Use process scheduling, backpressure, and buffering as the explanation path.

Generated Answer:
Explain the queue as a scheduling and backpressure layer that prevents the rest
of the system from being controlled by an external rate limit.
```

This is the difference between answering correctly and answering effectively.

## 7. Pre-Compiled Understanding Is Core

This discussion generalizes beyond this single project.

For almost every serious agent project, pre-compilation is a core capability.
The agent should not depend only on runtime retrieval from raw data.

Important pre-compiled layers may include:

```text
Project Understanding Layer
User Understanding Layer
Domain Understanding Layer
Organization Understanding Layer
Tool / Environment Layer
Evaluation Layer
Conversation History Layer
```

Runtime RAG is still useful, but it should not carry the full burden of
understanding. Runtime retrieval should operate over pre-compiled, structured,
maintained knowledge layers.

The principle can be stated as:

```text
Agent quality depends on pre-compiled understanding layers.
```

## 8. Relationship To The Project Understanding Layer

The Project Understanding Layer answers:

```text
What is true, likely, disputed, or unknown about the project?
```

The User Understanding Layer answers:

```text
How should this truth be explained to this user?
```

Together they form a cognitive bridge:

```text
Project Understanding Layer
+ User Understanding Layer
+ Query-Aware Context Assembly
+ Evidence-Grounded Reasoning
+ Belief Revision
= Personalized Project Archaeology Agent
```

## 9. Suggested Data Model

A first version of the user model could be:

```text
UserModel {
  known_domains[]
  weak_domains[]
  preferred_analogies[]
  rejected_analogies[]
  explanation_depth
  preferred_language
  reasoning_style
  active_goals[]
  accepted_claims[]
  disputed_claims[]
  corrections[]
  current_research_path[]
}
```

This model should be linked to the rest of the system:

```text
User correction
-> user evidence
-> affected user beliefs
-> answer strategy update
-> future context assembly changes
```

## 10. Governance And Safety

The Me Layer is powerful and sensitive. It must not become an invisible user
profile used to manipulate the user.

It should be a transparent collaboration memory.

Required properties:

- inspectable by the user
- editable by the user
- deletable by the user
- resettable by the user
- scoped by project or workspace when needed
- protected by strong privacy boundaries
- used only to improve collaboration and explanation quality

The system should be able to say, in effect:

```text
I remember you are familiar with X, so I will explain Y through that lens.
I remember you corrected Z, so I will not repeat the old explanation.
I know your current goal is A, so I will not over-expand into unrelated details.
```

This is different from an advertising profile or opaque behavioral model. It is
a user-controlled cognitive interface.

## 11. MVP Scope

The first version does not need a complex user modeling system.

A practical MVP can include:

```text
known_domains
active_goals
preferred_explanation_depth
preferred_language
accepted_user_corrections
rejected_explanations
useful_analogies
current_research_path
```

Minimal behavior:

- user corrections are persisted
- rejected explanations are not repeated
- useful analogies can be reused
- answers are adapted to known domains
- the user can inspect and edit the stored model

## 12. Conclusion

The project should not only compile knowledge about repositories. It should also
compile knowledge about the user.

The essential idea:

```text
Agent 不仅要提前编译项目，也要提前编译“我”。
真正有效的回答来自项目知识与用户认知结构之间的对齐。
```

This turns the system from a project understanding agent into a cognitive bridge
between a user and a project.
