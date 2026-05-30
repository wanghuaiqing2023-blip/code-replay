# Project Archaeology Memo

## 核心判断

这个项目的目标不是做一个普通的代码问答工具，也不是复刻 Understand-Anything 的代码结构图谱，而是构建一个面向 Git 仓库的历史理解系统。

它关注的问题是：

- 项目是如何演化成现在这样的？
- 哪些提交聚合成了关键历史阶段？
- 哪些设计意图隐藏在多个历史阶段之间的关系里？
- 项目的长期工程哲学是如何从历史演化中浮现出来的？

一句话定义：

> 从 Git 历史中抽取项目演化结构，并逐层抽象为设计意图和工程哲学的系统。

## 从 Understand-Anything 得到的启发

Understand-Anything 的重要启发是：好用的代码理解产品不能只靠聊天，必须先把仓库压缩成结构化上下文。

它的核心方法是：

- 先生成全局知识图谱
- 再让 LLM 基于图谱进行局部解释
- 图谱作为导航层，LLM 作为推理层

但它主要回答的是：

> 代码现在是什么样。

我们的项目更应该回答：

> 代码为什么变成现在这样。

因此，我们可以借鉴 Understand-Anything 的产品方法，但不能沿用它的核心图谱对象。它的中心是当前代码结构，我们的中心应该是历史演化结构。

## 核心概念模型

当前确定的抽象链条是：

```text
Commit
  -> Cluster
  -> Cluster Graph
  -> Evolution Graph
  -> Design Intent
  -> Project Philosophy
```

### Commit

真实 Git 提交，是最底层事实。

Commit 包含：

- hash
- author
- date
- message
- changed files
- diff summary
- parent relation

### Cluster

Cluster 是一组 commit 的事实聚合，表示一个阶段性变化或演化单元。

Cluster 不是设计决策本身，而是历史材料的压缩结果。它必须能回溯到真实 commits。

### Cluster Graph

Cluster Graph 是全局历史地图。

节点是 Cluster，边表示：

- 时间先后
- 依赖
- 承接
- 替代
- 回滚
- 稳定化
- 文档化
- 测试补齐

它提供项目的宏观历史视角。

### Evolution Graph

Evolution Graph 是从 Cluster Graph 中抽取出的有主题的关键演化子图。

例如：

```text
引入 Repository 抽象
  -> 迁移 Service 使用 Repository
  -> 删除 handler 中的 direct SQL
```

它不是预定义的 Pattern，而是从真实 Cluster Graph 中浮现出来的局部演化图。

### Design Intent

Design Intent 是对 Evolution Graph 的解释性抽象。

例如：

> 这条演化路径最合理地说明：作者希望隔离业务逻辑和持久化实现。

它不是事实，必须带有：

- supporting evidence
- counter evidence
- confidence
- uncertainty

### Project Philosophy

Project Philosophy 是多个 Design Intent 长期重复后呈现出的工程哲学。

例如：

- 偏好显式边界
- 偏好渐进迁移
- 偏好稳定核心、扩展边界
- 偏好简单实现而非高度抽象

## 关于 Subcluster 的结论

我们讨论过是否需要 Subcluster，最后决定去掉。

原因是：

- Subcluster 介于 Commit 和 Cluster 之间，事实依据不够稳定
- 它容易变成“LLM 觉得这些 commit 更像一组”
- 如果没有 PR、issue、branch 等天然边界，很容易制造牵强抽象

因此核心模型保留：

```text
Commit -> Cluster
```

如果未来大型 Cluster 内部确实需要进一步拆分，可以使用临时辅助字段，例如 `commit_groups`，但不进入核心概念体系。

## 关键产品原则

这个产品不应该一次性追求大而全。

更好的产品形态是：

> 先画出宏观地图，再让用户通过与 LLM 对话探索微观和细节。

也就是说：

- 系统先提供 Cluster Graph，帮助用户理解项目整体演化
- 用户从全局地图中选择感兴趣的阶段或路径
- 系统抽取相关 Evolution Graph
- LLM 围绕这个局部图解释设计意图、证据、反例和不确定性

这能避免两个问题：

- 一开始就让 LLM 盲目读全仓库
- 一开始就试图生成完整、封闭、不可探索的大报告

## 用户体验方向

用户接口可以是聊天窗口，但底层不应该是简单 RAG。

合理体验应该是：

1. 用户看到项目的全局历史地图
2. 用户选择或询问某个演化阶段
3. 系统定位相关 Cluster / Evolution Graph
4. LLM 给出局部设计解释
5. 用户继续追问证据、反例、替代方案和后续影响

典型问题包括：

- 这个项目经历过哪些重大演化阶段？
- 插件系统是怎么被引入的？
- 这个抽象为什么出现？
- 哪个阶段改变了项目方向？
- 某个设计后来是否被证明有效？
- 哪些设计被替换或回滚了？
- 作者长期偏好的工程哲学是什么？

## 与 Understand-Anything 的区别

两者相似之处：

- 都先结构化仓库
- 都提供全局视角
- 都结合 LLM 做局部解释
- 都需要证据引用和增量更新

根本区别：

```text
Understand-Anything:
理解当前代码结构。

本项目:
理解历史演化和设计意图。
```

Understand-Anything 的图谱回答：

> 项目现在是什么样。

我们的图谱回答：

> 项目为什么变成现在这样。

## 实现思路

MVP 可以先限制范围：

- 只支持 Git 仓库
- 暂不接 GitHub PR / issue
- 先支持一种语言或语言无关的 diff 分析
- 先输出 Cluster Graph
- 再支持对局部 Evolution Graph 的 LLM 分析

基础 pipeline：

```text
1. 读取 Git commit DAG
2. 提取每个 commit 的结构化特征
3. 将 commits 聚合为 clusters
4. 构建 Cluster Graph
5. 从 Cluster Graph 中抽取 Evolution Graph
6. 用 LLM 分析局部 Evolution Graph 的设计意图
7. 用户通过聊天继续探索
```

每个 LLM 结论都必须能追溯：

- 来自哪些 commit
- 来自哪些 diff
- 涉及哪些文件
- 支持证据是什么
- 反例是什么
- 置信度是多少

## 最重要的产品信念

这个系统的价值不在于一次性告诉用户所有答案，而在于提供一张可信的历史地图，让用户可以不断下钻。

好的产品不是一开始就追求大而全，而是：

> 先给用户宏观地图，再让用户通过与 LLM 对话，自己探索局部设计决策和细节。

