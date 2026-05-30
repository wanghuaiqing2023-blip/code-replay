# Githru Reference Analysis

## 1. Why Githru Matters

Githru is directly relevant to Code Replay because it focuses on understanding
software development history through Git metadata, rather than only explaining
the current code structure.

Reference materials:

- Repository: https://github.com/githru/githru
- Paper: `docs/githru.pdf`
- arXiv: https://arxiv.org/abs/2009.03115

The paper frames Git metadata as a source of historical context. This is aligned
with Code Replay's core premise:

```text
Git history is not just a log of changes.
It is the raw material from which project evolution, design intent, and
engineering philosophy can be reconstructed.
```

## 2. Githru's Core Contribution

Githru proposes an interactive visual analytics system for large Git commit
graphs. Its key technical contributions are:

```text
graph reconstruction
clustering
Context-Preserving Squash Merge, CSM
```

These techniques are used to abstract a large commit graph while preserving
important topology such as branches and merges.

Githru then provides:

- an overview of development history
- cluster-level summaries
- comparison between commit clusters
- file hierarchy views
- commit-level drilldown

This confirms one of Code Replay's product assumptions:

```text
Users should first get a trustworthy historical map, then drill down into
specific phases, paths, and design decisions.
```

## 3. What We Should Borrow

### 3.1 Git Topology Must Be Preserved

Githru does not treat commits as a flat collection of text documents. It
preserves the Git commit DAG and uses branch / merge topology as part of the
analysis.

Code Replay should do the same.

Do not build the first version as:

```text
all commits -> embedding -> arbitrary clusters
```

Instead, use:

```text
commit DAG
-> first-parent traversal
-> branch / merge handling
-> topology-aware clustering
```

### 3.2 Commit Similarity Features

Githru uses a simple but useful set of commit similarity features:

```text
author similarity
commit type similarity
changed file similarity
commit message similarity
commit date proximity
```

In the source implementation, author, commit type, and changed files are scored
with Jaccard similarity; commit messages use TF-IDF cosine similarity; commit
date is normalized by time distance.

For Code Replay, this suggests a first scoring model:

```text
CommitPairScore =
  w_file * file_jaccard
+ w_dir * directory_jaccard
+ w_message * message_similarity
+ w_author * author_similarity
+ w_type * commit_type_similarity
+ w_time * time_proximity
+ w_topology * topology_relation
```

The exact weights should be configurable because different repositories have
different commit hygiene and development styles.

### 3.3 Threshold-Based Cluster Granularity

Githru allows users to control clustering granularity through a threshold and
preference weights.

Code Replay should adopt this idea, but expose it in a more agent-friendly way:

```text
coarse
balanced
fine
```

Internally, those modes can map to threshold and weight presets.

### 3.4 Context-Preserving Squash Merge

The most important idea for our first version is Context-Preserving Squash
Merge.

A design change is often developed on a branch and merged back into the main
line. If we ignore branch commits, we lose context. If we naively include every
branch commit, the history map becomes noisy. If we squash too aggressively, we
lose evidence.

CSM suggests a better approach:

```text
preserve the merge point as the main visible node
collect relevant branch context into that node
retain access to the original source commits
```

For Code Replay, this becomes:

```text
merge commit / mainline commit
-> context-preserving synthetic commit
-> original branch commits remain evidence links
```

This should be treated as evidence compression, not evidence deletion.

### 3.5 Cluster Summary

Githru summarizes clusters with metadata such as:

```text
authors
commit types
keywords
modified files
modified directories
changed lines of code
file-to-author relation
```

Code Replay can extend this into an archaeology-focused cluster summary:

```text
cluster id
time span
commit count
representative commits
authors
dominant files / directories
keywords
change type distribution
churn
release / tag boundaries
merge context
evidence anchors
candidate evolution role
uncertainty
```

## 4. What We Should Not Copy Directly

### 4.1 Githru Is A Visual Analytics System

Githru is primarily designed for visual exploration. Code Replay is designed for
evidence-grounded conversation and design-history reconstruction.

Therefore, Githru's visualization ideas are useful, but the core output of Code
Replay should be:

```text
Cluster Graph
Evolution Graph
Design Intent
Evidence-backed Claim
Conversational drilldown
```

not only an interactive graph UI.

### 4.2 Githru Does Not Solve Design Intent

Githru helps users inspect and compare commit clusters. It does not infer,
validate, or revise design intent.

Code Replay must add:

```text
DesignClaim
supporting evidence
counter evidence
confidence
uncertainty
belief revision
```

### 4.3 Githru Is Not A Reusable Backend Library

The public repository is mainly a frontend implementation with sample JSON data.
The README does not provide a complete general-purpose pipeline for applying the
system to arbitrary repositories.

For Code Replay, the right approach is:

```text
study the paper and implementation
borrow the concepts and scoring features
reimplement a small Python/CLI pipeline suitable for Codex skill usage
```

## 5. Mapping To Code Replay

Githru maps into Code Replay as follows:

```text
Githru graph reconstruction
-> Code Replay commit DAG reconstruction

Githru clustering
-> Commit -> Cluster aggregation

Githru CSM
-> merge-aware evidence compression

Githru summary view
-> Cluster summary and historical overview

Githru comparison view
-> compare clusters / evolution phases

Code Replay extension
-> Evolution Graph, Design Intent, ClaimGraph, Belief Revision
```

The most important distinction:

```text
Githru:
Understand and explore development history metadata.

Code Replay:
Use development history as evidence to reconstruct design intent and project
philosophy.
```

## 6. First-Version Algorithm Proposal

The first version of the `project-archaeology` skill can use a Githru-inspired
pipeline:

```text
1. Extract Git commits.
2. Build the commit DAG.
3. Detect first-parent path, branches, merges, and tags.
4. Optionally apply CSM-style merge compression.
5. Compute pairwise similarity for topology-near commits:
   - file overlap
   - directory overlap
   - message similarity
   - author similarity
   - inferred commit type
   - time proximity
6. Cluster commits with threshold-based heuristics.
7. Build Cluster Graph.
8. Generate cluster summaries.
9. Extract topic-specific Evolution Graphs.
10. Ask the LLM to infer Design Intent with evidence and uncertainty.
```

The first version should avoid global semantic clustering. It should prioritize
traceability and topology preservation.

## 7. Skill Implementation Implications

The Codex App skill should include scripts such as:

```text
extract_git_history.py
score_commits.py
cluster_commits.py
build_cluster_graph.py
summarize_clusters.py
extract_evolution_graph.py
```

The skill instructions should explicitly require:

- do not summarize the repository directly
- first build or load the Git history abstraction
- treat clusters as candidate historical units, not final design decisions
- attach every design-intent explanation to commits, diffs, and files
- distinguish facts from inference
- report confidence and uncertainty

## 8. Open Design Questions

Several questions remain open:

- Should CSM be enabled by default?
- How should merge commits without clean branch context be handled?
- How should repositories with poor commit messages be clustered?
- Should tags/releases always cut clusters?
- Should cluster granularity be user-controlled or automatically selected?
- How should Cluster Graph connect to ClaimGraph?
- How should Project Philosophy be inferred without overgeneralization?

These should become early evaluation cases.

## 9. Conclusion

Githru should be treated as a primary technical reference for Code Replay's Git
history abstraction layer.

The most useful parts are:

```text
topology-preserving Git graph abstraction
commit similarity scoring
threshold-based clustering
Context-Preserving Squash Merge
cluster summary and comparison
```

Code Replay should not stop at Githru's visual analytics layer. It should build
on top of it:

```text
Git metadata
-> Cluster Graph
-> Evolution Graph
-> Design Intent
-> Evidence-backed Claim
-> Belief Revision
-> Personalized dialogue
```
