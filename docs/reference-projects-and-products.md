# 参考项目与产品形态清单

## 1. 分类原则

本项目需要研究两类对象：

```text
开源实现：
可以阅读源码，学习具体架构、数据结构、工具编排、评估方式。

非开源产品形态：
不能直接研究实现，但可以研究交互体验、能力边界、用户工作流和产品定位。
```

需要特别注意：

```text
Codex CLI 是开源项目，可以研究 openai/codex 的实现。
但 Codex 作为完整 OpenAI 产品，包括背后模型、托管执行环境和云端产品能力，并不是完整开源系统。
```

---

## 2. 开源实现可研究

### 2.1 Coding Agent / Agent Harness

#### openai/codex

GitHub: https://github.com/openai/codex

参考价值：

- terminal coding agent harness
- 工具调用
- sandbox / approval
- 终端交互
- 上下文管理
- patch / 文件编辑流程
- 本地 workspace agent 设计

边界：

- CLI / harness 代码可研究
- 背后模型和完整云端产品能力不可视为开源

#### SWE-agent

GitHub: https://github.com/SWE-agent/SWE-agent

参考价值：

- coding agent harness
- 工具使用策略
- 任务轨迹记录
- 与 SWE-bench 结合的评估方式
- 自动修复 bug 的执行链路

#### Aider

GitHub: https://github.com/Aider-AI/aider

参考价值：

- Git diff 驱动的代码修改
- patch 交互
- 多文件上下文管理
- commit 粒度
- 可回滚编辑流程

#### LangGraph

GitHub: https://github.com/langchain-ai/langgraph

参考价值：

- 状态机式 agent 编排
- graph-based workflow
- 可恢复执行
- 多步骤任务状态管理

#### AutoGen

GitHub: https://github.com/microsoft/autogen

参考价值：

- 多 agent 协作
- agent 消息协议
- 工具调用编排

#### CrewAI

GitHub: https://github.com/crewAIInc/crewAI

参考价值：

- role-based agent orchestration
- 多 agent 分工
- workflow abstraction

#### CodeFrame

GitHub: https://github.com/frankbria/codeframe

参考价值：

- Think -> Build -> Prove -> Ship 流程
- 显式 workflow
- 质量门禁
- human-in-the-loop
- agent adapter 思想

对本项目的启发：

```text
Ask -> Trace -> Reconstruct -> Verify -> Remember
```

---

### 2.2 Code Graph / 代码理解

#### codegraph

GitHub: https://github.com/colbymchenry/codegraph

参考价值：

- 代码语义图谱
- tree-sitter 解析
- 函数、类、调用、导入、继承关系
- callers / callees
- impact analysis
- 本地 SQLite / FTS 索引
- MCP 接口

对本项目的启发：

```text
CodeGraph 可以作为当前代码结构层。
我们需要在其上扩展 Git 历史、PR / issue、ClaimGraph 和 BeliefGraph。
```

#### tree-sitter

GitHub: https://github.com/tree-sitter/tree-sitter

参考价值：

- 多语言 AST 解析
- 函数、类、import、route、config 等实体抽取
- 代码结构索引基础设施

#### SCIP / LSIF

参考项目：

- https://github.com/sourcegraph/scip
- https://github.com/sourcegraph/lsif-go

参考价值：

- 跨语言代码索引格式
- 符号跳转
- 引用关系
- 大仓库代码导航

#### CodeQL

GitHub: https://github.com/github/codeql

参考价值：

- 结构化代码查询
- 语义模式识别
- 跨文件分析

#### Semgrep

GitHub: https://github.com/semgrep/semgrep

参考价值：

- 规则化代码模式匹配
- 轻量静态分析
- 多语言扫描

---

### 2.3 Belief Revision / 信念维护

#### mnemebrain-lite

GitHub: https://github.com/mnemebrain/mnemebrain-lite

参考价值：

- belief graph
- evidence provenance
- support / attack evidence
- confidence
- contradiction as first-class state
- Belnap 四值逻辑
- evidence-driven revision

对本项目的启发：

```text
不要在证据冲突时强行选一个答案。
需要允许 disputed / both / unresolved 状态。
```

#### ftl-beliefs

PyPI: https://pypi.org/project/ftl-beliefs/

参考价值：

- claim registry
- source / dependency / contradiction tracking
- IN / OUT / STALE 状态
- nogoods contradiction database
- stale 检测
- 面向 agent 的轻量 TMS 模型

对本项目的启发：

```text
ClaimGraph 可以先从轻量 TMS 模型开始。
不必一开始实现完整 AGM 或 NARS。
```

#### OpenNARS

GitHub: https://github.com/opennars/opennars

参考价值：

- 非公理化推理系统
- 不完整知识下的 belief revision
- truth value: frequency + confidence
- 经验驱动的知识更新

#### OpenNARS for Applications

GitHub: https://github.com/opennars/OpenNARS-for-Applications

参考价值：

- 更偏应用场景的 NARS 实现
- 可作为持续推理和 belief revision 的理论参考

#### Common Lisp Reasoner

Website: https://reasoner.sourceforge.net/

参考价值：

- 经典 TMS / ATMS
- dependency-directed reasoning
- justification-based truth maintenance
- assumption-based truth maintenance

---

### 2.4 Graph RAG / Evidence Graph

#### Microsoft GraphRAG

GitHub: https://github.com/microsoft/graphrag

参考价值：

- 从资料中抽取实体关系
- 构建图谱
- 社区摘要
- 多层检索
- graph-based RAG

#### LlamaIndex PropertyGraph

GitHub: https://github.com/run-llama/llama_index

参考价值：

- 属性图索引
- 图检索
- 向量检索结合
- structured retrieval

#### LightRAG

GitHub: https://github.com/HKUDS/LightRAG

参考价值：

- 轻量 graph-based RAG
- 高效检索组织
- 图和向量结合

#### Neo4j / Neo4j Vector

GitHub: https://github.com/neo4j/neo4j

参考价值：

- 图数据库
- 图查询
- 向量检索结合
- 适合作为后期复杂 ClaimGraph / EvidenceGraph 的候选基础设施

---

### 2.5 Benchmark / Evaluation

#### SWE-bench

GitHub: https://github.com/SWE-bench/SWE-bench

参考价值：

- 真实 GitHub issue 驱动的代码修复 benchmark
- patch 验证
- coding agent 评估方式

#### RepoBench

GitHub: https://github.com/Leolty/repobench

参考价值：

- 跨文件代码理解
- repo-level context evaluation

#### AgentBench

GitHub: https://github.com/THUDM/AgentBench

参考价值：

- 通用 agent 多任务评估
- 多步推理
- 工具使用能力

#### LongBench

GitHub: https://github.com/THUDM/LongBench

参考价值：

- 长上下文能力评估
- 长文档理解

#### RULER

GitHub: https://github.com/NVIDIA/RULER

参考价值：

- 长上下文评估
- 上下文窗口内信息定位能力

---

## 3. 非开源产品形态可研究

这些产品不应列入“开源项目”，但值得研究其产品形态、用户体验和能力边界。

### 3.1 Cursor

参考价值：

- IDE 内 agent 交互
- 代码上下文组装
- inline edit
- 用户和 agent 协作方式
- 局部代码导航体验

### 3.2 Windsurf

参考价值：

- agentic IDE workflow
- 多文件编辑体验
- 任务上下文维护
- 用户意图到代码操作的转换

### 3.3 Devin

参考价值：

- 长程任务执行
- 异步 agent
- 状态管理
- 计划、执行、反馈循环

### 3.4 Claude Code

参考价值：

- 命令行 / workspace agent 体验
- 对话式代码任务
- 工具执行链路
- 长程代码协作

### 3.5 GitHub Copilot

参考价值：

- IDE 深度集成
- PR / issue / repo 工作流
- 开发者日常使用场景

### 3.6 Sourcegraph Cody

参考价值：

- 大仓库代码搜索
- 企业代码库理解
- repo-level context
- 代码图谱和语义检索产品形态

---

## 4. Karpathy LLM Knowledge Base / LLM Wiki

这不是单一开源项目，但对本项目有重要启发。

核心模式：

```text
raw sources -> LLM compilation -> structured wiki -> query via index and links
```

对本项目的对应关系：

```text
raw/
-> GitHub raw evidence

wiki/
-> project understanding layer

schema / CLAUDE.md
-> domain harness rules

lint
-> consistency check / belief revision

index
-> query router / context map

log
-> revision log / audit trail
```

本项目应该升级为：

```text
LLM Wiki + Evidence Graph + ClaimGraph + Belief Revision
```

---

## 5. 推荐研究优先级

### 第一优先级：核心实现参考

```text
openai/codex
codegraph
tree-sitter
ftl-beliefs
mnemebrain-lite
Microsoft GraphRAG
LightRAG
```

原因：

- 覆盖 agent harness
- 覆盖代码事实层
- 覆盖信念维护
- 覆盖图谱检索

### 第二优先级：架构与理论参考

```text
LangGraph
SWE-agent
Aider
OpenNARS
LlamaIndex PropertyGraph
SCIP / LSIF
CodeQL
Semgrep
```

原因：

- 补充 workflow、评估、代码索引和推理机制

### 第三优先级：产品形态研究

```text
Cursor
Windsurf
Devin
Claude Code
GitHub Copilot
Sourcegraph Cody
```

原因：

- 研究用户体验、交互形态、能力边界
- 不作为开源实现参考

---

## 6. 对本项目的直接映射

```text
CodeGraph / tree-sitter / SCIP
-> CodeGraphDB / 当前代码结构层

Git history + GitHub API
-> HistoryDB / CollaborationDB

GraphRAG / LightRAG / PropertyGraph
-> EvidenceGraph / 多层检索

ftl-beliefs / mnemebrain-lite / TMS
-> ClaimGraph / Belief Revision

openai/codex / SWE-agent / LangGraph
-> Agent Harness / Tool Orchestration / State Machine

Karpathy LLM Wiki
-> Knowledge Compilation / Project Understanding Layer

SWE-bench / RepoBench / AgentBench
-> Evaluation Harness
```

---

## 7. 结论

本项目不应该寻找一个可以直接照搬的开源项目。更现实的方式是从多个方向吸收成熟机制：

```text
coding agent harness
代码图谱
GitHub 历史证据
Graph RAG
ClaimGraph
Belief Revision
LLM Knowledge Compilation
Evaluation Harness
```

最终目标不是复刻某个项目，而是组合出一个新的系统：

```text
面向 GitHub 项目设计历史理解的证据驱动型 agent harness。
```
