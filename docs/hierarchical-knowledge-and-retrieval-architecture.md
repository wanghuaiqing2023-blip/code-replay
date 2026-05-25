# 分层知识与检索架构设计笔记

## 1. 背景

本项目的目标不是简单总结 GitHub 仓库，而是构建一个能够持续理解、追踪、修正项目设计历史的智能体。

最近几轮讨论揭示出一个关键方向：

```text
项目知识不能被平铺进一个向量库。
它应该被组织成分层、多维、可路由、可追溯、可维护的知识体系。
```

这个体系的目的不是为了抽象本身，而是为了更高效、更准确地回答用户问题。

用户每次提问通常只和项目知识库中的一小部分信息有关。系统应该判断：

```text
这个问题需要查哪些层？
这个问题需要查哪些维度？
这个问题需要访问哪些数据库？
这个问题需要组装哪些 context？
```

最终目标是构建一个 query-aware retrieval harness。

---

## 2. 从微观到宏观的分层信息模型

项目知识天然存在粒度层级。系统不应该把代码片段、commit、PR、设计决策、架构总结混在同一层检索。

建议至少划分为 6 层：

```text
L0 Raw Evidence
原始证据层：
commit diff、PR comment、issue comment、code snippet、doc paragraph

L1 Atomic Entities
原子实体层：
function、class、file、config key、dependency、route、test case

L2 Local Context
局部上下文层：
文件职责、函数调用链、相关测试、文件历史

L3 Change Event
变化事件层：
某个 PR、commit cluster、重构事件、依赖替换、API 变更

L4 Design Claim
设计判断层：
这个模块用于 X；这个抽象是为了解决 Y；这个依赖替换是因为 Z

L5 Belief / Architecture
当前理解层：
模块地图、架构原则、设计时间线、关键决策、演化阶段
```

这个分层的价值在于：不同问题命中不同层。

示例：

```text
“这个函数做什么？”
-> L1 / L2

“这个文件最近为什么改？”
-> L0 / L2 / L3

“为什么引入 queue？”
-> L3 / L4，并回溯 L0 证据

“这个项目的架构是怎么演化的？”
-> L5 / L4 / L3，然后只下钻关键 L0 证据

“你之前说这是性能原因，有证据吗？”
-> L4 claim -> supporting evidence L0
```

---

## 3. 分层 Context 组装

Context 组装不应该是把 top-k 检索结果直接塞给模型，而应该是结构化组装。

一个回答所需 context 可以表示为：

```text
ContextPack {
  question
  intent
  scope
  layer_plan
  beliefs[]
  claims[]
  change_events[]
  evidence[]
  raw_snippets[]
  uncertainty[]
}
```

常见组装顺序：

```text
Question
Task scope
Relevant belief summary
Relevant claims
Key change events
Supporting / contradicting evidence
Raw snippets only when necessary
Conversation memory
```

核心原则：

```text
模型拿到的应该是组织好的材料，而不是一堆无结构碎片。
```

---

## 4. 从层级到多维抽象

宏观/微观只是粒度维度。项目知识还可以从其他维度抽象归纳。

同一条证据可以同时属于多个维度。例如一个 PR 既可以是：

```text
粒度维度：Change Event
时间维度：某个演化阶段的转折点
因果维度：为了解决某个问题
社会维度：由某个 maintainer 推动
风险维度：引入了兼容性破坏
设计维度：形成了新的模块边界
```

因此，我们应该把项目知识建模为多维空间，而不是单一树状层级。

---

## 5. 推荐的多维抽象维度

### 5.1 时间维度

识别项目演化阶段：

```text
原型期
快速扩张期
稳定化期
性能优化期
架构重构期
兼容性维护期
```

支持的问题：

- 项目什么时候从实验性变成稳定？
- 哪些阶段发生过架构转向？
- 哪些模块是在压力出现后才被抽象出来的？

### 5.2 因果维度

把信息组织成：

```text
Problem -> Constraint -> Decision -> Tradeoff -> Consequence
```

这是设计历史还原的核心维度。

示例：

```text
问题：请求阻塞
约束：第三方 API rate limit
决策：引入 queue
代价：一致性复杂度上升
后果：worker 模块出现
```

### 5.3 设计意图维度

按工程意图分类：

```text
correctness
performance
scalability
maintainability
security
developer experience
compatibility
observability
cost
product requirement
```

这个维度有助于回答：

```text
为什么不是另一种设计？
```

### 5.4 结构维度

描述架构形态：

```text
module boundary
dependency direction
API surface
data flow
control flow
ownership boundary
runtime topology
storage model
extension point
```

### 5.5 社会协作维度

很多设计决策来自人和组织，而不只是技术。

```text
author
reviewer
maintainer
team ownership
disagreement
review tension
rejected proposal
governance rule
release pressure
```

设计过程往往隐藏在 review comment 和 issue discussion 中。

### 5.6 稳定性维度

判断某个理解的生命周期：

```text
experimental
accepted
disputed
deprecated
superseded
removed
revived
```

一个架构判断不是永远有效，它可能被后续证据修正或推翻。

### 5.7 证据强度维度

信息不是平权的。证据强度应该显式建模：

```text
direct statement in PR
design doc
issue discussion
review comment
commit message
code diff
inferred from structure
user correction
```

回答时应该优先使用强证据，并标注弱证据或推断。

### 5.8 影响范围维度

判断某个改动的影响范围：

```text
local function
file
module
API
storage
runtime behavior
developer workflow
public compatibility
architecture
```

这个维度可以帮助识别真正的设计事件。

### 5.9 反事实维度

记录没有采用的方案：

```text
rejected alternative
abandoned abstraction
reverted approach
postponed design
failed migration
```

设计决策通常不只是选择了 A，而是放弃了 B/C。

### 5.10 用户理解维度

系统不仅维护项目知识，也维护用户研究状态：

```text
user already knows
user doubts
user corrected
user cares about
user wants executive summary
user wants implementation detail
```

同一个项目，对不同用户应该组装不同 context。

---

## 6. Project Understanding Lattice

可以把这套机制命名为：

```text
Project Understanding Lattice
```

它区别于单纯的 Retrieval Pyramid。

```text
Retrieval Pyramid:
解决粒度问题

Understanding Lattice:
解决多维理解问题
```

一个知识项可以表示为：

```text
KnowledgeItem {
  granularity: raw / entity / event / claim / belief
  time: phase / date / release
  intent: performance / maintainability / security / ...
  structure: module / API / dataflow / runtime / ...
  causality: problem / constraint / decision / tradeoff / consequence
  social: author / reviewer / disagreement / ownership
  evidence_strength: direct / indirect / inferred
  stability: accepted / disputed / deprecated / superseded
  impact_scope: file / module / API / architecture
}
```

这样检索就不再是单一 top-k，而是多维过滤、路由和组装。

---

## 7. 多维抽象的真正目的：高效回答用户问题

多维抽象不是为了建立复杂知识体系，而是为了让系统知道：

```text
这个问题应该查哪里，不应该查哪里。
```

如果系统维护了多个数据库：

```text
code graph
git history
PR / issue index
review comments
docs index
claim graph
belief snapshots
user memory
evaluation traces
```

每个问题通常只需要其中几个。

示例：

```text
“这个函数是做什么的？”
需要：
code graph
file context
maybe tests

不需要：
完整 PR / issue 历史
```

```text
“为什么引入 queue？”
需要：
git history
PR / issue index
claim graph
相关 code graph

不一定需要：
全仓库文档
```

```text
“你之前关于 queue 的理解是不是错了？”
需要：
claim graph
contradiction records
belief snapshots
user corrections
supporting evidence

不需要：
重新扫描全代码
```

---

## 8. Query Routing Layer

系统需要一个 Query Routing Layer，把用户问题转换为检索计划。

输入：

```text
User question
Conversation context
Current project state
Known beliefs
```

输出：

```text
QueryUnderstanding {
  intent
  target
  granularity
  time_need
  evidence_need
  history_need
  consistency_need
  user_memory_need
}
```

进一步生成：

```text
RetrievalPlan {
  databases_to_query[]
  indexes_to_use[]
  max_depth
  context_budget
  required_evidence_types[]
  fallback_path[]
}
```

示例：

```text
Question: 为什么引入 queue？

Intent: design_motivation
Target: queue
Granularity: module / design claim
History_need: high
Evidence_need: high

Databases:
  - code_graph
  - git_history
  - github_prs
  - github_issues
  - claim_graph
  - belief_snapshots
```

---

## 9. 数据库职责划分

不建议做一个大而全的库。应该按职责拆分。

```text
CodeGraphDB
当前代码结构、符号、调用、依赖

HistoryDB
commit、diff、文件演化、rename、release

CollaborationDB
PR、issue、review、discussion、作者、争议

EvidenceDB
标准化后的证据对象

ClaimDB
原子 claim、support、contradiction、dependency

BeliefDB
当前理解、架构视图、设计时间线、版本快照

UserMemoryDB
用户纠错、偏好、已确认理解、当前研究路径
```

Query Router 决定查哪些库，而不是默认全查。

---

## 10. 最小充分 Context

Context 组装目标不是塞最多信息，而是组装最小充分信息。

定义：

```text
Minimal Sufficient Context
= 回答该问题所需的最小证据集
+ 必要的上层理解
+ 必要的反证或不确定性
```

收益：

1. 降低 token 成本
2. 降低无关信息干扰
3. 提高回答可验证性
4. 提高检索和生成速度
5. 降低幻觉风险

---

## 11. 常见问题类型与默认检索计划

第一版可以支持 8 类 intent：

```text
code_explanation
module_summary
design_motivation
evolution_timeline
evidence_check
contradiction_check
architecture_overview
user_memory_followup
```

默认数据库选择：

```text
code_explanation:
  code_graph, docs, tests

module_summary:
  code_graph, docs, claim_graph

design_motivation:
  history, github_prs, github_issues, claim_graph

evolution_timeline:
  history, github_prs, release_notes, change_events

evidence_check:
  claim_graph, evidence_db, raw evidence

contradiction_check:
  claim_graph, evidence_db, belief_snapshots, user_memory

architecture_overview:
  belief_db, claim_graph, module_graph, key_events

user_memory_followup:
  user_memory, belief_snapshots, relevant claims
```

---

## 12. Karpathy 的 LLM Knowledge Base / LLM Wiki 启发

Karpathy 提出的 LLM Knowledge Base / LLM Wiki 模式对本项目有直接启发。

其核心思路可以概括为：

```text
raw sources -> LLM compilation -> structured wiki -> query via index and links
```

这区别于普通 RAG：

```text
raw documents -> chunks -> embeddings -> top-k retrieval
```

关键启发是：

```text
知识不应该只是被检索，也应该被持续编译、维护和校验。
```

---

## 13. 对本项目的启发

### 13.1 LLM 作为 Knowledge Compiler

我们不能每次用户提问都从 commit、PR、issue、diff 中重新推理一遍。

应该持续把原始资料编译成中间知识层：

```text
raw evidence
-> entities
-> change events
-> design claims
-> belief snapshots
-> project wiki / decision map
```

LLM 不只是 answer generator，而是 knowledge compiler。

### 13.2 Raw Source 必须保留为事实源

事实源包括：

```text
git commits
diffs
PRs
issues
review comments
docs
code snapshots
release notes
user corrections
```

派生层包括：

```text
module summaries
change events
design claims
architecture timelines
belief snapshots
```

任何派生结论都不能脱离 raw evidence。

### 13.3 Schema 比 Wiki 本身更重要

我们需要的不是普通知识库，而是带操作规范的知识库。

需要定义：

```text
Evidence schema
Claim schema
Belief schema
Revision schema
Query routing schema
Context assembly schema
Lint rules
```

例如，任何 design claim 必须包含：

```text
claim type
scope
supporting evidence
contradicting evidence
confidence
dependency claims
status
```

### 13.4 需要 Lint / Maintenance

LLM 生成的知识层必须被维护。

我们的 lint 不只是格式检查，而是理解一致性检查：

```text
claim 没有证据 -> 标记 unsupported
claim 和新证据冲突 -> 标记 disputed
claim 依赖的上游失效 -> 标记 stale
同一模块有多个冲突职责描述 -> 触发合并或修订
旧回答引用 rejected claim -> 标记过期
```

这对应我们系统中的 Consistency Daemon。

### 13.5 人类可读知识层仍然有价值

Markdown wiki 的优势是：

- 人类可读
- 可版本控制
- 可审计
- 适合作为对话 context
- 适合团队协作

建议采用混合形态：

```text
结构化数据库：
EvidenceDB / ClaimDB / BeliefDB / RevisionLog

人类可读投影：
project-wiki/
  architecture.md
  modules/auth.md
  decisions/queue-introduction.md
  timelines/api-evolution.md
  open-questions.md
```

数据库负责查询、一致性维护和依赖传播。

Markdown wiki 负责人类阅读、对话上下文和审计。

---

## 14. 我们对 LLM Wiki 模式的升级

Karpathy 的 LLM Wiki 更适合个人或团队知识管理。

本项目面对 GitHub 项目历史，会遇到更多复杂性：

- 大量低质量 commit
- 时间关系非常关键
- PR / issue / review 多源冲突
- 同一 claim 可能被后续设计推翻
- 私有仓库需要权限边界
- 多用户可能有不同理解状态

因此不能只做 Markdown wiki。需要升级为：

```text
LLM Wiki + Evidence Graph + ClaimGraph + Belief Revision
```

对应关系：

```text
raw/
-> GitHub raw evidence

wiki/
-> project understanding layer

CLAUDE.md schema
-> domain harness rules

lint
-> consistency check / belief revision

index.md
-> query router / context map

log.md
-> revision log / audit trail
```

---

## 15. 推荐系统链路

综合上述讨论，推荐系统链路为：

```text
Raw Evidence Ingestion
-> Evidence Normalization
-> Layered Indexing
-> Multi-dimensional Tagging
-> Knowledge Compilation
-> ClaimGraph / BeliefGraph Update
-> Consistency Lint
-> Query Routing
-> Targeted Retrieval
-> Minimal Context Assembly
-> Evidence-grounded Answer
-> Memory / Belief Revision
```

这条链路体现了四个核心原则：

1. 信息分层
2. 多维抽象
3. 按问题路由
4. 持续编译和维护知识层

---

## 16. MVP 建议

第一版不需要实现全部维度。建议先实现：

### 16.1 必备层级

```text
Raw Evidence
Atomic Entity
Change Event
Design Claim
Belief Snapshot
```

### 16.2 必备维度

```text
granularity
time
causality
evidence_strength
```

### 16.3 必备数据库

```text
CodeGraphDB
HistoryDB
CollaborationDB
EvidenceDB
ClaimDB
BeliefDB
UserMemoryDB
```

### 16.4 必备 Query Intent

```text
code_explanation
design_motivation
evolution_timeline
evidence_check
contradiction_check
architecture_overview
```

### 16.5 必备维护机制

```text
claim without evidence -> unsupported
new contradiction -> disputed
upstream claim rejected -> downstream stale
user correction -> evidence + revision trigger
old answer using stale claim -> marked outdated
```

---

## 17. 核心结论

本项目的信息架构应该从普通 RAG 升级为：

```text
Layered Retrieval + Multi-dimensional Abstraction + Knowledge Compilation + Belief Revision
```

核心目标不是维护一个复杂知识库，而是更高效地回答用户问题：

```text
按问题选择维度
按维度选择数据库
按数据库选择索引
按回答目标组装最小充分 context
```

最重要的一句话：

```text
不要把项目知识当成一堆可检索文本。
要把它当成一个可编译、可路由、可验证、可修订的项目理解系统。
```
