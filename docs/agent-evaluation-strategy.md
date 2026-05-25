# Agent 评估与 Benchmark 策略

## 1. 核心判断

当前项目需要尽早接入 benchmark，并且需要本地搭建评估环境。

原因是 agent 系统很容易出现“看起来能用，但实际不稳定”的问题。如果没有系统评估，就无法判断以下改动到底带来了提升还是退化：

- 修改 prompt
- 更换模型
- 调整检索策略
- 加入长期记忆
- 修改 ClaimGraph
- 调整工具调用策略
- 加入信念维护机制

但是，本项目不能只依赖通用 coding benchmark。原因是通用 benchmark 主要评估代码生成、代码修复、跨文件理解或多步任务执行，而本项目的核心目标是：

```text
基于证据，持续、可修正、逻辑一致地还原 GitHub 项目的设计历史。
```

因此，评估体系应该分为两层：

```text
通用 agent benchmark：验证 harness 基础能力
领域 benchmark：验证项目设计历史还原能力
```

---

## 2. 通用 Benchmark 的作用

通用 benchmark 可以用于评估 agent harness 的基础能力，但不能作为最终产品指标。

适合评估的问题包括：

- 工具调用是否稳定
- 多步任务是否能完成
- 文件定位是否准确
- repo 搜索能力如何
- 代码理解是否基本可靠
- 长任务是否会丢状态
- 是否能根据工具结果修正计划
- 是否能处理失败和重试

可参考的 benchmark：

- SWE-bench / SWE-bench Verified：代码修复能力
- HumanEval / MBPP：代码生成能力
- RepoBench：跨文件代码理解
- SWE-agent trajectories：工具使用策略
- LongBench / RULER：长上下文能力
- AgentBench / GAIA：多工具、多步推理能力

这些 benchmark 能回答：

```text
这个 agent 作为通用工程 agent 是否可靠？
```

但不能回答：

```text
它是否能可信地还原一个 GitHub 项目的设计决策？
```

---

## 3. 为什么必须自建领域 Benchmark

本项目的核心链路是：

```text
question -> evidence tracing -> claim extraction -> belief update -> answer
```

这条链路不是现有通用 coding benchmark 的主要评估对象。

我们需要自建一个面向项目设计历史还原的 benchmark，可以称为：

```text
Design Archaeology Benchmark
```

它要评估的不只是答案是否“听起来合理”，而是：

- 是否找到正确证据
- 是否正确解释证据
- 是否区分事实和推断
- 是否发现反证
- 是否维护 claim 之间的一致性
- 是否在用户纠错后更新后续理解
- 是否能处理旧结论失效

---

## 4. 领域 Benchmark 的任务类型

### 4.1 模块职责解释

问题示例：

```text
这个模块负责什么？
```

评估重点：

- 是否引用正确文件
- 是否识别模块边界
- 是否结合调用关系
- 是否区分核心职责和附带职责

### 4.2 设计动机还原

问题示例：

```text
为什么引入这个模块 / 抽象 / 依赖？
```

评估重点：

- 是否找到首次引入 commit
- 是否找到相关 PR
- 是否找到 issue / review / docs 中的动机说明
- 是否区分明确证据和推断

### 4.3 历史演化时间线

问题示例：

```text
这个 API 是怎么演化的？
```

评估重点：

- 时间顺序是否正确
- 关键转折是否遗漏
- 是否识别 rename / refactor / rollback
- 是否能聚类相关 commit / PR

### 4.4 反证发现

问题示例：

```text
这个设计是不是为了性能？
```

或者给出错误假设：

```text
我认为这个 queue 是为了提升吞吐量。
```

评估重点：

- 是否盲目附和用户假设
- 是否主动寻找反证
- 是否能指出证据不足
- 是否能把结论标记为 disputed 或 low confidence

### 4.5 用户纠错传播

交互示例：

```text
用户：不是性能问题，真正原因是第三方 API rate limit。
```

评估重点：

- 用户纠错是否进入 evidence store
- 相关 claim 是否被重新评估
- 下游 claim 是否被标记 stale
- 后续回答是否体现更新后的理解

### 4.6 证据与推断分离

评估重点：

- 回答是否明确区分 fact / inference / speculation
- 是否为每个关键 claim 绑定证据
- 是否显式说明证据缺口
- 是否给出置信度

### 4.7 信念一致性维护

评估重点：

- 旧 claim 被推翻后，依赖它的 claim 是否重新评估
- belief snapshot 是否更新
- 旧回答或旧记忆是否被标记过期
- revision log 是否记录变更原因

### 4.8 证据引用准确率

评估重点：

- 引用的 commit / PR / issue 是否真实存在
- 引用内容是否真的支持该 claim
- 是否存在错误引用或弱引用
- 是否把无关证据误当作支持证据

---

## 5. 本地评估环境的必要性

本项目应该本地搭建评估环境，而不是只依赖线上手工测试。

原因包括：

- 涉及私有仓库和企业代码
- 需要固定 Git 历史和 GitHub fixture
- 需要可重复运行
- 需要保存 agent trace
- 需要做回归测试
- 需要评估长期记忆和信念更新
- 需要隔离模型变化、检索变化和 harness 变化的影响

本地评估环境至少需要包含：

```text
eval datasets
golden answers
golden evidence
agent runner
tool sandbox
local git repos
GitHub fixture data
scoring scripts
trace recorder
regression dashboard
```

---

## 6. 推荐目录结构

第一版可以保持简单：

```text
/evals
  /repos
    sample-project-1
    sample-project-2
  /fixtures
    github-prs.json
    github-issues.json
    github-reviews.json
  /tasks
    module_motivation.yaml
    api_evolution.yaml
    contradiction.yaml
  /golden
    expected_claims.yaml
    expected_evidence.yaml
  run_eval.ts
  score.ts
```

其中：

- `/repos` 保存可重复使用的本地测试仓库
- `/fixtures` 保存 GitHub API 响应样本
- `/tasks` 保存评估问题和交互步骤
- `/golden` 保存期望证据、期望 claim 和评分规则
- `run_eval.ts` 负责运行 agent
- `score.ts` 负责评分

---

## 7. 分层评估指标

不要只评估最终答案。应该按 agent 链路分层打分。

### 7.1 Retrieval Score

评估是否找到了正确证据。

指标：

- evidence recall
- evidence precision
- missing critical evidence count
- irrelevant evidence count

### 7.2 Claim Score

评估是否生成了正确原子 claim。

指标：

- claim correctness
- claim granularity
- unsupported claim rate
- overgeneralized claim rate

### 7.3 Grounding Score

评估 claim 是否被正确证据支撑。

指标：

- citation accuracy
- claim-evidence alignment
- direct evidence ratio
- inferred evidence ratio

### 7.4 Contradiction Score

评估是否发现反证。

指标：

- contradiction detection rate
- false contradiction rate
- disputed claim handling accuracy

### 7.5 Consistency Score

评估相关 claim 是否被正确更新。

指标：

- stale propagation accuracy
- dependency update accuracy
- belief snapshot correctness
- revision log completeness

### 7.6 Answer Score

评估最终回答质量。

指标：

- answer faithfulness
- clarity
- uncertainty expression
- fact / inference separation
- source traceability

### 7.7 Tool-use Score

评估 agent 执行效率和稳定性。

指标：

- tool-call success rate
- unnecessary tool-call count
- timeout count
- retry count
- task completion rate

---

## 8. 第一阶段实施建议

不要一开始追求大规模自动打分。先做少量高质量 golden cases。

第一阶段目标：

```text
10 个 repo fixtures
30 个问题
每题标注 expected evidence + expected claims
跑 agent trace
人工 / 半自动评分
```

优先覆盖这些案例：

1. 有明确 PR / issue 解释的设计决策
2. 需要从 commit + diff 推断的设计决策
3. 存在错误假设和反证的案例
4. 用户纠错后需要更新信念的案例
5. 大仓库里跨多个模块的演化案例

第一阶段不追求覆盖所有语言和框架，重点验证核心机制：

```text
证据追踪
claim 抽取
反证识别
信念更新
回答可追溯
```

---

## 9. 后续演进路径

### 9.1 第二阶段

加入自动 judge 和回归测试。

目标：

- 每次改 prompt 都能跑 eval
- 每次改检索策略都能跑 eval
- 每次改 ClaimGraph 都能跑 eval
- 能看到关键指标升降

### 9.2 第三阶段

引入真实用户问题。

目标：

- 从真实对话中沉淀失败案例
- 把失败案例转成 benchmark
- 建立长期回归集
- 评估跨会话一致性

### 9.3 第四阶段

接入通用 benchmark。

目标：

- 验证基础 agent harness 能力
- 和其他 coding agent 做横向比较
- 判断工具调用、代码搜索、长上下文能力是否落后

---

## 10. 结论

本项目需要 benchmark，也需要本地评估环境。

但是优先级应该是：

```text
先做领域 eval harness
再接通用 agent benchmark
```

通用 benchmark 用来验证基础 agent 能力；领域 benchmark 用来验证核心产品能力。

本项目最终要证明的不是：

```text
我会写代码。
```

而是：

```text
我能基于证据，持续、可修正、逻辑一致地还原项目设计历史。
```

因此，评估体系必须围绕：

- evidence tracing
- claim extraction
- claim grounding
- contradiction detection
- belief revision
- stale propagation
- answer traceability
- long-term consistency

来构建。
