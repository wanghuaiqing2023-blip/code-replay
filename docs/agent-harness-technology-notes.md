# Agent Harness 技术体系笔记

## 1. 核心判断

Harness 不是外围胶水代码，而是把大模型能力转化为真实系统能力的关键工程层。

大模型本身提供的是语言理解、推理、生成和模式识别能力；harness 负责把这些能力接入真实场景，并让它在工具、状态、权限、验证、记忆和失败恢复中稳定工作。

可以用下面的方式理解二者关系：

```text
Model:
理解语言、生成计划、推理可能性

Harness:
定义任务边界、提供工具、维护状态、执行动作、验证结果、处理错误、记录证据
```

因此，严肃的 agent 系统不能只是：

```text
LLM + prompt + API
```

更合理的形态是：

```text
LLM + domain harness + tools + state + verification + memory + trace
```

对于“GitHub 项目设计历史还原智能体”来说，harness 应该是核心技术资产。

---

## 2. Harness 填补的关键鸿沟

### 2.1 从语言到行动

用户的自然语言目标通常是模糊的，例如：

```text
帮我理解这个模块为什么这么设计
```

模型只能给出可能的推理。Harness 必须把它转化为可执行流程：

```text
identify module
inspect current code
trace file history
find PR / issue / review
extract candidate claims
verify evidence
answer with confidence
```

### 2.2 从上下文窗口到长期状态

模型上下文有限，不能自然维护长期项目理解。Harness 需要维护：

- 当前 repo
- 当前研究目标
- 已查证据
- 已确认 claim
- 被推翻 claim
- 用户纠错
- belief snapshot
- 未解问题

### 2.3 从合理回答到可验证回答

模型容易生成合理叙述，但合理不等于可信。Harness 必须强制：

- 每个结论绑定证据
- 没有证据时降低置信度
- 存在反证时进入 disputed 状态
- 相关下游结论被标记 stale
- 回答能够回溯到 evidence / claim / belief

### 2.4 从一次性问答到可恢复任务

真实任务会中断、失败、继续、重算。Harness 需要提供：

- task state
- trace log
- checkpoint
- resumability
- partial result
- retry policy
- stale state detection

### 2.5 从通用智能到领域能力

通用模型不天然知道“GitHub 设计历史还原”应该如何工作。Harness 要把领域方法固化为流程：

```text
Ask -> Trace -> Claim -> Verify -> Revise -> Remember
```

---

## 3. 普适 Harness 技术分类

### 3.1 上下文管理

上下文压缩只是上下文管理的一部分。完整上下文管理包括：

- context summarization：压缩历史对话和工具结果
- working memory：当前任务短期状态
- long-term memory：跨会话持久记忆
- episodic memory：一次任务轨迹
- semantic memory：稳定知识
- context packing：把最相关信息装入上下文窗口
- context eviction：决定丢弃什么
- salience scoring：判断哪些信息重要
- citation-aware compression：压缩时保留证据引用
- hierarchical summaries：项目级、模块级、文件级多层摘要

对于我们的项目，关键要求是：

```text
压缩不能丢失证据链。
```

### 3.2 工具调用与执行编排

工具调用是 agent 从“会说”变成“会做”的核心。

关键技术包括：

- tool registry
- tool schema
- tool selection
- tool call planning
- tool result normalization
- parallel tool calls
- retry / backoff
- timeout control
- permission / approval
- sandboxing
- idempotency
- side-effect classification
- tool audit log

在我们的项目中，工具不仅包括 shell，还包括：

- Git tools
- GitHub API
- code search
- CodeGraph
- tree-sitter parser
- ClaimGraph
- evidence retriever
- consistency checker

### 3.3 任务状态机

复杂任务不能只靠 prompt。需要显式状态机。

通用状态可能是：

```text
created
planning
gathering_context
executing
verifying
blocked
completed
failed
stale
```

对于项目设计历史还原，可以改成：

```text
question_received
scope_identified
evidence_tracing
claim_extraction
consistency_checking
answering
memory_updated
```

状态机的价值是：

- 可恢复
- 可观察
- 可调试
- 可评估
- 可中断后继续

### 3.4 计划与反思机制

计划与反思不是让模型无限“思考”，而是把计划、执行、检查分开。

关键技术包括：

- task decomposition
- plan generation
- plan revision
- self-check
- critic / verifier
- reflection after failure
- escalation policy
- stopping criteria

通用模式：

```text
Plan -> Act -> Observe -> Revise -> Verify
```

我们的领域模式：

```text
Trace Plan -> Evidence Collection -> Claim Draft -> Consistency Check -> Answer
```

### 3.5 检索与记忆系统

RAG 是基础，但 agent harness 中的检索与记忆必须更工程化。

关键技术包括：

- vector search
- keyword search
- graph search
- hybrid retrieval
- recency-aware retrieval
- authority-aware retrieval
- provenance tracking
- memory write policy
- memory consolidation
- memory invalidation
- contradiction detection

对我们的项目，最重要的是：

```text
retrieval + provenance + invalidation
```

没有 invalidation，长期记忆会被错误结论污染。

### 3.6 验证闭环

验证闭环是决定 agent 可靠性的核心。

Coding agent 的验证通常是：

```text
build / test / lint / typecheck
```

我们的验证闭环应该是：

```text
claim 是否有证据
claim 是否被反证攻击
claim 是否和已有 belief 冲突
claim 依赖是否有效
```

可用技术包括：

- factuality check
- source verification
- consistency check
- constraint check
- schema validation
- contradiction search
- human confirmation
- confidence recalculation
- dependency validity check

### 3.7 轨迹记录与可观测性

没有 trace，agent 难以调试，也难以获得用户信任。

关键技术包括：

- reasoning trace
- tool trace
- evidence trace
- decision trace
- token / cost / latency trace
- failure trace
- answer-to-source mapping
- replay
- state diff
- revision history

对我们的项目来说，trace 不只是内部调试工具，也是产品能力。用户需要看到：

```text
你为什么这么说？
这个结论来自哪些证据？
哪些地方只是推断？
哪些旧理解被更新了？
```

### 3.8 错误恢复与鲁棒性

真实工具会失败，模型会走偏，任务会中断。

关键技术包括：

- checkpoint
- resumability
- retries
- fallback tools
- partial result handling
- stale state detection
- interruption recovery
- failure classification
- human handoff
- rollback
- supersede

对我们的项目，尤其需要：

- belief revision rollback
- claim supersede
- answer invalidation
- stale propagation

### 3.9 权限、安全与副作用控制

只要 agent 能调用工具，就必须控制副作用。

关键技术包括：

- sandbox
- approval gate
- read / write separation
- dangerous operation detection
- secret redaction
- least privilege
- audit log
- data boundary
- tenant isolation
- network access policy

即使我们的项目主要读取 GitHub 数据，也会涉及：

- 私有仓库
- 企业代码
- 用户纠错记录
- 内部架构理解
- 组织级知识资产

因此，权限和数据边界是基础能力。

### 3.10 评估 Harness

没有评估，agent 无法稳定进步。

关键技术包括：

- task benchmark
- golden traces
- regression suite
- judge model
- human review
- adversarial cases
- ablation testing
- hallucination rate
- tool-use success rate
- citation accuracy
- memory consistency score

我们项目的评估指标可以包括：

- 证据引用准确率
- claim 支撑充分性
- 反证发现率
- stale propagation 正确率
- 用户纠错后更新是否生效
- 长对话前后一致性
- 设计决策还原是否可追溯

---

## 4. 对本项目最关键的 Harness 组合

我们的系统不应该只是普通 agent，而应该是一个面向 GitHub 项目历史理解的 domain harness。

核心链路应该是：

```text
context management
-> tool orchestration
-> evidence retrieval
-> claim extraction
-> consistency verification
-> belief revision
-> traceable answer
-> memory update
```

可以拆成以下几个内部 harness：

### 4.1 Evidence Harness

管理所有证据来源：

```text
git commits
diffs
PRs
issues
review comments
docs
code symbols
user corrections
```

职责：

- 采集证据
- 标准化证据
- 记录来源
- 计算权威性
- 支持证据检索
- 支持证据版本化

### 4.2 Reasoning Harness

把原始证据转换成结构化 claim：

```text
evidence -> claim candidates -> support/contradiction links -> confidence
```

职责：

- 抽取候选结论
- 拆分原子 claim
- 绑定证据
- 标记推断深度
- 计算置信度
- 区分 fact / inference / speculation

### 4.3 Consistency Harness

负责信念维护：

```text
changed evidence
-> affected claims
-> stale propagation
-> local re-evaluation
-> revised belief snapshot
```

职责：

- 发现冲突
- 标记 stale
- 传播影响范围
- 重算局部理解
- 生成 revision log
- 维护 supersede 关系

### 4.4 Conversation Harness

负责长期对话体验：

```text
user question
-> retrieve relevant beliefs
-> identify uncertainty
-> trace missing evidence
-> update project memory
```

职责：

- 维护用户当前研究上下文
- 记录用户纠错
- 保存已确认结论
- 保留未解问题
- 支持跨会话继续

### 4.5 Evaluation Harness

负责系统质量改进：

```text
test questions
expected evidence
answer faithfulness
contradiction handling
stale update correctness
user correction propagation
```

职责：

- 构建 benchmark
- 记录 golden trace
- 做回归测试
- 评估证据引用准确率
- 评估信念更新正确性
- 评估长对话一致性

---

## 5. 优先级排序

如果按通用 agent 系统的重要性排序：

1. 工具调用与执行编排
2. 上下文管理和压缩
3. 任务状态机
4. 检索与记忆
5. 验证闭环
6. 轨迹记录和可观测性
7. 错误恢复
8. 权限和副作用控制
9. 计划 / 反思机制
10. 评估 harness

如果按本项目的重要性排序：

1. Evidence Harness
2. ClaimGraph / BeliefGraph
3. Consistency Harness
4. Context and Memory Harness
5. Tool Orchestration
6. Trace and Observability
7. Evaluation Harness
8. Permission and Data Boundary
9. Conversation Harness
10. Code Modification Harness

其中 Code Modification Harness 应该靠后。我们的核心不是写代码，而是维护可信的项目设计理解。

---

## 6. MVP 建议

第一版不应该实现完整 coding agent，而应该实现一个最小 domain harness。

MVP 需要包含：

```text
Tool registry
Repo workspace
Git / GitHub evidence tools
Evidence store
Claim store
Claim dependencies
Contradiction records
Belief snapshots
Trace log
Answer composer
```

最小流程：

```text
User asks question
-> identify repo/module/file scope
-> retrieve current code context
-> trace git/GitHub history
-> create evidence records
-> extract atomic claims
-> link supports / contradicts
-> compute confidence
-> update belief snapshot
-> answer with evidence and uncertainty
```

第一版的硬性原则：

- 不允许无来源结论进入长期记忆
- 用户纠错必须成为 evidence
- 被反证攻击的 claim 必须进入 disputed 或 stale
- 回答必须能追溯到 evidence
- 压缩必须保留证据链

---

## 7. 结论

Harness 是把模型能力变成可靠系统能力的部分。

对于这个项目，真正的壁垒不在“调用一个更强模型”，而在：

- 领域对象模型
- 证据结构
- 状态机
- 工具编排
- 信念维护
- 验证闭环
- 评估体系
- 用户纠错反馈
- 项目长期记忆

因此，这个项目更准确的定位不是：

```text
一个会分析 GitHub 的 LLM 应用
```

而是：

```text
一个面向 GitHub 项目历史理解的 agent harness，
LLM 是其中的推理引擎。
```

一句话总结：

```text
上下文压缩只是 harness 的一部分。
真正普适的 harness 技术，是把模型输出纳入一个有状态、有工具、有验证、有权限、有恢复能力的工程系统。
```
