<div align="center">

<h1>The Evolution of Attention in Large Language Models: Mechanisms, Trade-offs, and Emerging Trends</h1>

<p><strong>Zhentao Tan, Jingyi Shen, Yanbo Li, Yao Liu, Yue Wu, Jieping Ye</strong><br>
Alibaba Token Hub, Alibaba Group</p>

[![Paper](https://img.shields.io/badge/Paper-arXiv-b31b1b.svg)](https://arxiv.org/pdf/2609.39661)
[![Awesome](https://awesome.re/badge.svg)](#综述索引)
[![128 Papers](https://img.shields.io/badge/Papers-128-yellow.svg)](#综述索引)
[![Contributions Welcome](https://img.shields.io/badge/Contribution-Welcome-brightgreen.svg)](#参与贡献)
[![Citation](https://img.shields.io/badge/Citation-BibTeX-blue.svg)](#引用)

</div>

<div align="center">

[English](./README.md) | **中文版**

[Full Text](https://arxiv.org/abs/2609.39661)

一份持续更新的高效序列架构综述与结构化文献图谱，从**记忆中心视角**解释这些架构如何演进。

</div>

## 概述


本仓库是综述《[The Evolution of Attention in Large Language Models: Mechanisms, Trade-offs, and Emerging Trends](https://arxiv.org/abs/2609.39661)》的配套资源。

本文以**模型内部的上下文记忆**为统一概念，考察从 Softmax 注意力和稀疏注意力到线性注意力、状态空间模型及混合设计的高效序列架构。面对快速涌现的新架构，本文追问一个共同问题：这些设计表面各异，是否都在解决同一个核心问题——如何管理模型自身的上下文记忆？

为了连接通常被分别研究的不同方向，本文提出五维分析视角：

- **记忆表示（Memory Representation）**——上下文以何种形式存储
- **记忆更新（Memory Update）**——新信息如何写入
- **访问（Access）**——如何选择已存储的记忆
- **读出（Readout）**——如何取回所选记忆
- **集成（Integration）**——如何将读出结果重新融入计算

<div align="center">

![五维记忆中心框架](./assets/attention-primer.png)

</div>

基于 14 个主要模型谱系的 59 条发布级记录，以及对 11 个高性能开放权重模型的结构化比较，分析表明，记忆处理正日益呈现跨时间与跨网络深度的协同。

这一框架为比较当今架构提供了共同语言，也指向基于有状态、多维和选择性路由记忆的未来设计。

## 核心框架


不同序列架构通过表面上差异显著的算子向当前计算提供历史上下文：Softmax 注意力保留可单独寻址的记忆单元，稀疏注意力限制实际检查的单元，而循环机制则持续把历史压缩到一个或多个状态中。本文从功能层面把它们统一视为维护和使用**上下文记忆**的系统。上下文记忆是模型处理当前序列时可用的、模型内部且依赖输入的信息，包括 token KV、压缩 token 或片段、记忆槽、关联矩阵、结构化循环状态及其异构组合，但不包括预训练参数和外部检索语料库。

令 $t$ 表示当前序列位置，$x_t$ 表示记忆机制的当前输入，$q_t$ 表示用于查询上下文记忆的表示。记忆模式 $\rho$ 规定表示的类型、组织方式、粒度、容量和持久性。

**1. 记忆表示（Memory Representation）。** 模型维护的记忆 $\mathcal{M}_t$ 必须属于该记忆模式允许的状态空间：

<div align="center">

$$
\mathcal{M}_t \in \mathfrak{M}_{\rho}.
$$

</div>

模式可以描述持续增长的 token KV 列表、固定容量的摘要槽位、关联矩阵、结构化循环状态或多条异构记忆路径。因此，记忆表示既考察历史覆盖范围，也考察单个历史单元能否保持独立可寻址。

**2. 记忆更新（Memory Update）。** 令 $\mathcal{M}_t^{-}$ 和 $\mathcal{M}_t^{+}$ 分别表示处理当前输入之前和之后的记忆，则更新抽象为：

<div align="center">

$$
\mathcal{M}_t^{+}=\mathrm{Update}_{\rho}\!\left(\mathcal{M}_t^{-},x_t\right).
$$

</div>

该算子统一涵盖只追加的 KV 缓存、向固定槽位压缩、循环状态转移、衰减、Delta 校正以及受控擦除—写入规则。它描述记忆如何变化，而不决定查询最终使用其中哪些部分。

**3. 访问（Access）。** 令 $\widehat{\mathcal{M}}_t$ 表示暴露给读取路径的记忆视图；根据机制不同，它可以是当前更新之前或之后的状态。访问操作构造可用视图 $\mathcal{C}_t$：

<div align="center">

$$
\mathcal{C}_t=\mathrm{Access}_{\rho}\!\left(q_t,\widehat{\mathcal{M}}_t\right).
$$

</div>

结果可以包含全部因果 token 记忆、局部或路由选择的 token/块子集、一个或多个循环状态接口，以及后续读出所需的路由元数据。

**4. 读出（Readout）。** 读出决定查询如何从可访问视图中提取上下文信息：

<div align="center">

$$
r_t=\mathrm{Readout}_{\rho}\!\left(q_t,\mathcal{C}_t\right).
$$

</div>

典型实现包括归一化查询—键聚合、关联状态收缩和结构化状态投影。因此，访问回答“**哪些信息可以被读取**”，读出则回答“**如何对可用信息进行加权、解码或聚合**”。

**5. 集成（Integration）。** 集成把一个或多个已完成的读出转换为记忆模块的输出：

<div align="center">

$$
o_t=\mathrm{Integration}_{\rho}\!\left(r_t;x_t\right).
$$

</div>

其中 $r_t$ 可以是单个向量，也可以是一组来自不同注意力头、分支或记忆路径的读出。集成方式包括输出投影、拼接、求和、门控、条件路由以及异构记忆路径之间的融合。

这些公式定义的是功能角色，而非所有架构都必须遵循的统一计算图。同一操作可以同时承担多个角色：表示方式的变化可能改变访问，输入条件状态转移可能同时改变表示和更新，分层稀疏索引也可能同时影响地址表示与候选选择。

<a id="classification-logic-and-family-mapping"></a>

## 分类逻辑与研究路线映射


综述分类和五维分析框架承担不同作用。五条研究路线依据各自的核心技术问题与历史演进组织，并不是由五个维度机械推导而来。每项工作根据其核心贡献与技术谱系归入一条主要研究路线；少量跨领域方法在对另一设计方向具有实质贡献时，也会作为辅助索引出现在第二张表中。因此，本列表包含 128 项独立工作和 131 条分类记录。五维标签以多标签方式描述各方法具体修改的记忆功能，从而支持不同研究路线之间的横向比较。

**P** 表示该维度通常是该研究路线的主要设计目标，**S** 表示该维度通常继承自基础机制，或只在部分子类中被修改。该映射是解释性指引而非穷尽式统计；五个维度仍适用于每一条研究路线。

| 研究路线 | 记忆载体 | 记忆表示 | 记忆更新 | 访问 | 读出 | 集成 |
|---|---|:---:|:---:|:---:|:---:|:---:|
| Softmax 注意力 | 显式记忆单元 | **P** | **P** | **S** | **S** | **S** |
| 稀疏注意力 | 显式 token/块 KV | **S** | **S** | **P** | **S** | **S** |
| 线性注意力 | 关联循环状态 | **P** | **P** | **S** | **S** | **S** |
| 状态空间模型 | 结构化循环状态 | **P** | **P** | **S** | **S** | **S** |
| 混合架构 | 异构记忆载体 | **P** | **S** | **S** | **S** | **S** |

Softmax 注意力主要通过表示与更新减少显式记忆冗余或重组显式记忆。稀疏注意力通常保留 token/块级内容记忆和 Softmax 读出，同时把候选构造与访问预算分配作为核心访问问题。线性注意力与状态空间模型聚焦循环状态如何表示和更新；访问、读出和集成主要在路由、门控或多状态变体中变得更加显式。

混合架构需要不同的解释，因为它组合异构机制或访问模式，而不是增加一种独立的记忆算子。因此，记忆表示是其主要的家族级轴线；它是否进一步修改更新、访问、读出或集成，则取决于组合发生在层、头、分支还是 token 粒度。

<a id="architecture-landscape-and-future-directions"></a>

## 架构格局与未来方向


公开披露的 LLM 注意力设计呈现**多样化而非收敛**的趋势。在覆盖 14 个主要模型谱系的 59 条发布级记录中，58 条可分类：32 条采用单一主要机制，26 条属于混合架构；混合记录从 2022–2023 年的 0/6 上升到 2026 年的 17/27。多数混合设计仍采用预先设定的层级调度，仅有 5 条记录明确跨层复用 KV 表示、索引或 Top-k 决策。下方的高性能模型快照进一步体现了这种共存：11 个开放权重模型端点覆盖完整 GQA、稀疏注意力、潜在压缩和循环—显式混合等多种结构，但全部保留显式 token 检索路径。这些观察用于描述技术采用情况，并不把模型质量归因于某一种机制。

**高性能开放权重模型端点的注意力结构。** 快照采集于 2026 年 9 月 22 日；模型规模为总参数量，括号内为已公开的激活参数量。

| 模型端点 | 首次公开 | 模型规模 | 注意力结构 |
|---|:---:|:---:|---|
| MiMo-V2.6-Pro | 2026.09 | 1.02T（激活 42B） | 60 层 SWA-GQA + 10 层全局 GQA |
| DeepSeek-V4.1-Flash（max） | 2026.09 | 552B（预填充激活 8B；解码激活 16B） | CSA2 与 SWA；跨层复用 KV、索引和 Top-k |
| K2-Horizon-375B-A23B | 2026.09 | 375B（激活 23B） | 完整 GQA |
| GLM-5.3-Flash | 2026.08 | 320B（激活 18B） | 34 层 KDA + 11 层压缩索引器 DSA |
| GLM-5.3（max） | 2026.08 | 744B（激活 40B） | 基于 MLA 的 DSA，使用 IndexShare/IndexCache |
| Qwen3.8-Flash-Next | 2026.08 | 176B；主体 125B（激活 6B） | 每 3 层 GDN 配置 1 层 QSA |
| DeepSeek-V4 Pro 0813（max） | 2026.08 | 1.6T（激活 49B） | CSA/HCA 层级混合 |
| Qwen3.8-2.4T-A95B | 2026.08 | 2.4T（激活 95B） | 每 3 层 Gated DeltaNet 配置 1 层门控完整 GQA |
| Qwen3.8-27B（xhigh） | 2026.08 | 27B | 每 3 层 Gated DeltaNet 配置 1 层门控完整 GQA |
| Kimi K3（max） | 2026.07 | 2.8T（激活 104B） | 每 3 层 KDA 配置 1 层 Gated MLA |
| MiniMax-M3 | 2026.06 | 428B（激活 23B） | 3 层稠密注意力 + 57 层 MSA |

### 三层综合

本文并不把架构演进视为一系列算子的替代，而是将其理解为对上下文记忆不断扩展的重新设计：从机制层面的功能控制，走向架构层面的协同，并进一步延伸至未来的有状态记忆系统。

> 🔵 **机制层面——控制范围持续扩展**
>
> 显式记忆与状态方法保留不同的记忆接口，但都将设计控制扩展到日益重叠的记忆功能。

> 🔵 **架构层面——沿网络深度组织记忆**
>
> 层级组合把互补的记忆处理分布到不同表示阶段，跨层复用则延长选定记忆与路由产物的生命周期；二者共同使网络深度成为组织上下文记忆的新维度。

> 🔵 **前瞻层面——有状态多维记忆路由**
>
> 深度视角与时间范围、载体类型和表示粒度相结合，为在持久上下文记忆上协同进行稀疏写入与稀疏读取提供了动机。

<a id="literature-index"></a>

## 综述索引


- [Softmax 注意力](#softmax-attention)
  - [记忆表示效率](#memory-representation-efficiency)
  - [序列表示压缩](#sequence-representation-compression)
  - [读出与集成调制](#readout-and-integration-modulation)
- [稀疏注意力](#sparse-attention)
  - [结构约束稀疏注意力](#structure-constrained-sparse-attention)
  - [自路由稀疏注意力](#self-routing-sparse-attention)
  - [辅助代理稀疏路由](#auxiliary-proxy-sparse-routing)
  - [时间与跨层复用](#temporal-and-cross-layer-reuse)
- [线性注意力](#linear-attention)
  - [记忆更新规则](#memory-update-rules)
  - [容量扩展](#capacity-expansion)
  - [时间扩展](#temporal-expansion)
  - [辅助协调与路由](#auxiliary-coordination-and-routing)
- [状态空间模型](#state-space-models)
  - [状态动力学与选择性控制](#state-dynamics-and-selective-control)
  - [状态组织与读写细化](#state-organization-and-read-write-refinements)
- [混合架构](#hybrid-architecture)
  - [层级混合](#layer-wise-hybrid)
  - [头级混合](#head-wise-hybrid)
  - [分支级混合](#branch-wise-hybrid)
  - [token 级混合](#token-wise-hybrid)


<a id="softmax-attention"></a>

## Softmax 注意力


Softmax 注意力保留可枚举的显式记忆单元和归一化读出，其主要设计方向包括降低单 token 表示成本、压缩长历史，以及调制注意力得分和多头输出的集成方式。

<a id="memory-representation-efficiency"></a>

### 记忆表示效率

该类别在保留 Softmax 精确读出的前提下降低每个 token 的 KV 表示成本，涵盖从 MHA 经 GQA 到 MQA 的头共享谱系、低维潜在 KV 通道压缩，以及跨层 KV 共享。

- [Attention Is All You Need](https://proceedings.neurips.cc/paper/7181-attention-is-all-you-need) ![](https://img.shields.io/badge/arXiv-2017.06-red) ![](https://img.shields.io/badge/NeurIPS-2017-yellow)
- [Fast Transformer Decoding: One Write-Head is All You Need](https://arxiv.org/abs/1911.02150) ![](https://img.shields.io/badge/arXiv-2019.11-red)
- [GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://aclanthology.org/2023.emnlp-main.298/) ![](https://img.shields.io/badge/arXiv-2023.05-red) ![](https://img.shields.io/badge/EMNLP-2023-yellow)
- [DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model](https://arxiv.org/abs/2405.04434) ![](https://img.shields.io/badge/arXiv-2024.05-red)
- [TransMLA: Multi-Head Latent Attention Is All You Need](https://arxiv.org/abs/2502.07864) ![](https://img.shields.io/badge/arXiv-2025.02-red)
- [Reducing Transformer Key-Value Cache Size with Cross-Layer Attention](https://arxiv.org/abs/2405.12981) ![](https://img.shields.io/badge/arXiv-2024.05-red)

<a id="sequence-representation-compression"></a>

### 序列表示压缩

该类别压缩随序列长度增长的历史表示，包括将历史反复整合到固定容量循环记忆中，以及保留近期细节、逐渐粗化远端表示的动态分辨率记忆。

- [Transformer-XL: Attentive Language Models Beyond a Fixed-Length Context](https://arxiv.org/abs/1901.02860) ![](https://img.shields.io/badge/arXiv-2019.01-red)
- [Compressive Transformers for Long-Range Sequence Modelling](https://arxiv.org/abs/1911.05507) ![](https://img.shields.io/badge/arXiv-2019.11-red)
- [Recurrent Memory Transformer](https://arxiv.org/abs/2207.06881) ![](https://img.shields.io/badge/arXiv-2022.07-red)
- [TransformerFAM: Feedback attention is working memory](https://arxiv.org/abs/2404.09173) ![](https://img.shields.io/badge/arXiv-2024.04-red)
- [Trellis: Learning to Compress Key-Value Memory in Attention Models](https://arxiv.org/abs/2512.23852) ![](https://img.shields.io/badge/arXiv-2025.12-red)
- [Lattice: Learning to Compress the Cache in the Attention](https://research.google/pubs/lattice-learning-to-compress-the-cache-in-the-attention/) ![](https://img.shields.io/badge/Google_Research-2025-yellow)
- [Kwai Summary Attention Technical Report](https://arxiv.org/abs/2604.24432) ![](https://img.shields.io/badge/arXiv-2026.04-red)
- [DeepSeek-V4: Towards Highly Efficient Million-Token Context Intelligence](https://arxiv.org/abs/2606.19348) ![](https://img.shields.io/badge/arXiv-2026.06-red)

<a id="readout-and-integration-modulation"></a>

### 读出与集成调制

该类别不更换底层记忆集合，而是调整注意力读出和多头集成，包括在值聚合前变换得分或注意力图的调制方式，以及在集成阶段选择或重标定头读出的头路由与输出门控。

- [Talking-Heads Attention](https://arxiv.org/abs/2003.02436) ![](https://img.shields.io/badge/arXiv-2020.03-red)
- [Improving Transformers with Dynamically Composable Multi-Head Attention](https://arxiv.org/abs/2405.08553) ![](https://img.shields.io/badge/arXiv-2024.05-red)
- [Differential Transformer](https://arxiv.org/abs/2410.05258) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Forgetting Transformer: Softmax Attention with a Forget Gate](https://arxiv.org/abs/2503.02130) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [MoH: Multi-Head Attention as Mixture-of-Head Attention](https://arxiv.org/abs/2410.11842) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Gated Attention for Large Language Models: Non-linearity, Sparsity, and Attention-Sink-Free](https://arxiv.org/abs/2505.06708) ![](https://img.shields.io/badge/arXiv-2025.05-red)

<a id="sparse-attention"></a>

## 稀疏注意力


稀疏注意力保留细粒度 KV 候选，但限制每个查询实际读取的支持集。分类重点是稀疏支持集从哪里产生、以何种粒度组织，以及路由结果是否被复用。

<a id="structure-constrained-sparse-attention"></a>

### 结构约束稀疏注意力

该类别用稳定结构限制查询能够访问的 KV 对，包括训练前确定拓扑的架构预设模式和从训练后稠密模型规律中提炼的后验发现模式。

- [Generating Long Sequences with Sparse Transformers](https://arxiv.org/abs/1904.10509) ![](https://img.shields.io/badge/arXiv-2019.04-red)
- [Longformer: The Long-Document Transformer](https://arxiv.org/abs/2004.05150) ![](https://img.shields.io/badge/arXiv-2020.04-red)
- [Big Bird: Transformers for Longer Sequences](https://proceedings.neurips.cc/paper/2020/hash/c8512d142a2d849725f31a9a7a361ab9-Abstract.html) ![](https://img.shields.io/badge/arXiv-2020.07-red) ![](https://img.shields.io/badge/NeurIPS-2020-yellow)
- [LongNet: Scaling Transformers to 1,000,000,000 Tokens](https://arxiv.org/abs/2307.02486) ![](https://img.shields.io/badge/arXiv-2023.07-red)
- [PowerAttention: Exponentially Scaling of Receptive Fields for Effective Sparse Attention](https://arxiv.org/abs/2503.03588) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [Efficient Streaming Language Models with Attention Sinks](https://arxiv.org/abs/2309.17453) ![](https://img.shields.io/badge/arXiv-2023.09-red)
- [LM-Infinite: Zero-Shot Extreme Length Generalization for Large Language Models](https://arxiv.org/abs/2308.16137) ![](https://img.shields.io/badge/arXiv-2023.08-red)
- [MInference 1.0: Accelerating Pre-filling for Long-Context LLMs via Dynamic Sparse Attention](https://arxiv.org/abs/2407.02490) ![](https://img.shields.io/badge/arXiv-2024.07-red)

<a id="self-routing-sparse-attention"></a>

### 自路由稀疏注意力

该类别利用查询和上下文本身生成稀疏候选集，涵盖动态 token 分组、块聚合与选择，以及摘要 token 路由。

- [Reformer: The Efficient Transformer](https://arxiv.org/abs/2001.04451) ![](https://img.shields.io/badge/arXiv-2020.01-red) ![](https://img.shields.io/badge/ICLR-2020-yellow)
- [Efficient Content-Based Sparse Attention with Routing Transformers](https://arxiv.org/abs/2003.05997) ![](https://img.shields.io/badge/arXiv-2020.03-red) ![](https://img.shields.io/badge/TACL-2021-yellow)
- [Quest: Query-Aware Sparsity for Efficient Long-Context LLM Inference](https://arxiv.org/abs/2406.10774) ![](https://img.shields.io/badge/arXiv-2024.06-red)
- [XAttention: Block Sparse Attention with Antidiagonal Scoring](https://arxiv.org/abs/2503.16428) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [MoBA: Mixture of Block Attention for Long-Context LLMs](https://arxiv.org/abs/2502.13189) ![](https://img.shields.io/badge/arXiv-2025.02-red)
- [Optimizing Mixture of Block Attention](https://arxiv.org/abs/2511.11571) ![](https://img.shields.io/badge/arXiv-2025.11-red)
- [Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention](https://aclanthology.org/2025.acl-long.1126/) ![](https://img.shields.io/badge/arXiv-2025.02-red) ![](https://img.shields.io/badge/ACL-2025-yellow)
- [InfLLM-V2: Dense-Sparse Switchable Attention for Seamless Short-to-Long Adaptation](https://arxiv.org/abs/2509.24663) ![](https://img.shields.io/badge/arXiv-2025.09-red)
- [DashAttention: Differentiable and Adaptive Sparse Hierarchical Attention](https://arxiv.org/abs/2605.18753) ![](https://img.shields.io/badge/arXiv-2026.05-red)
- [COBS: Cumulant Order Block Sparse Attention](https://arxiv.org/abs/2607.09052) ![](https://img.shields.io/badge/arXiv-2026.07-red)
- [Landmark Attention: Random-Access Infinite Context Length for Transformers](https://arxiv.org/abs/2305.16300) ![](https://img.shields.io/badge/arXiv-2023.05-red)
- [Simplified Sparse Attention via Gist Tokens](https://arxiv.org/abs/2604.20920) ![](https://img.shields.io/badge/arXiv-2026.04-red)
- [Hierarchical Sparse Attention Done Right: Toward Infinite Context Modeling](https://arxiv.org/abs/2607.02980) ![](https://img.shields.io/badge/arXiv-2026.07-red)

<a id="auxiliary-proxy-sparse-routing"></a>

### 辅助代理稀疏路由

该类别通过轻量辅助索引器估计候选支持集，包括对单个 token 条目排序的 token 粒度代理路由和对上下文块或压缩条目排序的块与压缩条目代理路由。

- [TokenButler: Token Importance is Predictable](https://arxiv.org/abs/2503.07518) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [DeepSeek-V3.2: Pushing the Frontier of Open Large Language Models](https://arxiv.org/abs/2512.02556) ![](https://img.shields.io/badge/arXiv-2025.12-red)
- [LongCat Sparse Attention: Taming the Lightning via Streaming-aware Hierarchical Cross-Layer Indexing](https://arxiv.org/abs/2608.01662) ![](https://img.shields.io/badge/arXiv-2026.08-red)
- [SAS: Simple Attention Sparsification via End-to-End Optimization of Context Ranking](https://arxiv.org/abs/2609.13141) ![](https://img.shields.io/badge/arXiv-2026.09-red)
- [HashAttention: Semantic Sparsity for Faster Inference](https://arxiv.org/abs/2412.14468) ![](https://img.shields.io/badge/arXiv-2024.12-red)
- [HATA: Trainable and Hardware-Efficient Hash-Aware Top-k Attention for Scalable Large Model Inference](https://arxiv.org/abs/2506.02572) ![](https://img.shields.io/badge/arXiv-2025.06-red) ![](https://img.shields.io/badge/Findings_of_ACL-2025-yellow)
- [SeerAttention: Learning Intrinsic Sparse Attention in Your LLMs](https://arxiv.org/abs/2410.13276) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [SeerAttention-R: Sparse Attention Adaptation for Long Reasoning](https://arxiv.org/abs/2506.08889) ![](https://img.shields.io/badge/arXiv-2025.06-red)
- [SpotAttention: Plug-In Block-Sparse Routing for Pretrained Long-Context Transformers](https://arxiv.org/abs/2606.22874) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [MiniMax Sparse Attention](https://arxiv.org/abs/2606.13392) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [MiniMax-M3](https://huggingface.co/MiniMaxAI/MiniMax-M3) ![](https://img.shields.io/badge/Model_Card-2026-yellow)
- [On the Design of Qwen3.8-Next Architecture: Evaluation, Efficiency, and Training Stability](https://arxiv.org/abs/2608.30320) ![](https://img.shields.io/badge/arXiv-2026.08-red)
- [Qwen3.8-Flash-Next Technical Report and Model Card](https://github.com/QwenLM/Qwen3.8-Flash-Next) ![](https://img.shields.io/badge/Technical_Report-2026-yellow)
- [DeepSeek-V4: Towards Highly Efficient Million-Token Context Intelligence](https://arxiv.org/abs/2606.19348) ![](https://img.shields.io/badge/arXiv-2026.06-red)

<a id="temporal-and-cross-layer-reuse"></a>

### 时间与跨层复用

该类别通过复用已有稀疏支持集来摊薄索引成本，包括在相邻查询或解码步骤间的时间支持集复用和沿网络深度的跨层索引复用。

- [Recall Before You Rank: Similarity-Guided Top-$K$ Reuse for Efficient Long-Context Attention](https://arxiv.org/abs/2607.27692) ![](https://img.shields.io/badge/arXiv-2026.07-red)
- [TidalDecode: Fast and Accurate LLM Decoding with Position Persistent Sparse Attention](https://arxiv.org/abs/2410.05076) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Kascade: A Practical Sparse Attention Method for Long-Context LLM Inference](https://arxiv.org/abs/2512.16391) ![](https://img.shields.io/badge/arXiv-2025.12-red)
- [IndexCache: Accelerating Sparse Attention via Cross-Layer Index Reuse](https://arxiv.org/abs/2603.12201) ![](https://img.shields.io/badge/arXiv-2026.03-red)
- [GLM-5.2](https://huggingface.co/zai-org/GLM-5.2) ![](https://img.shields.io/badge/Model_Card-2026-yellow)
- [GLM-5.3](https://huggingface.co/zai-org/GLM-5.3) ![](https://img.shields.io/badge/Model_Card-2026-yellow)
- [You Only Index Once: Cross-Layer Sparse Attention with Shared Routing](https://arxiv.org/abs/2606.06467) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [DeepSeek-V4.1-Flash: Pushing the Limits of KV Cache Compression](https://arxiv.org/abs/2609.19969) ![](https://img.shields.io/badge/arXiv-2026.09-red)
- [HySparse: A Hybrid Sparse Attention Architecture with Oracle Token Selection and KV Cache Sharing](https://arxiv.org/abs/2602.03560) ![](https://img.shields.io/badge/arXiv-2026.02-red)
- [HySparse2: Hybrid Sparse Attention with Two-Level KV Sharing](https://arxiv.org/abs/2609.26368) ![](https://img.shields.io/badge/arXiv-2026.09-red)

<a id="linear-attention"></a>

## 线性注意力


线性注意力以一个或多个循环关联状态替代可枚举的 token 缓存。经典形式采用与上下文长度无关的状态，后续变体则通过扩展、分区或组织多个状态来增加容量与时间覆盖范围。其演进主线包括可控状态编辑、容量扩展、时间范围扩展，以及辅助协调与路由。

<a id="memory-update-rules"></a>

### 记忆更新规则

该类别研究关联状态如何保留、擦除和写入，涵盖累加与校正更新（含 Delta 式校正）、保留与门控状态编辑，以及基于损失的在线优化。

- [Transformers are RNNs: Fast Autoregressive Transformers with Linear Attention](https://proceedings.mlr.press/v119/katharopoulos20a.html) ![](https://img.shields.io/badge/arXiv-2020.06-red) ![](https://img.shields.io/badge/ICML-2020-yellow)
- [Linear Transformers Are Secretly Fast Weight Programmers](https://arxiv.org/abs/2102.11174) ![](https://img.shields.io/badge/arXiv-2021.02-red)
- [Parallelizing Linear Transformers with the Delta Rule over Sequence Length](https://arxiv.org/abs/2406.06484) ![](https://img.shields.io/badge/arXiv-2024.06-red)
- [DeltaProduct: Improving State-Tracking in Linear RNNs via Householder Products](https://arxiv.org/abs/2502.10297) ![](https://img.shields.io/badge/arXiv-2025.02-red)
- [Retentive Network: A Successor to Transformer for Large Language Models](https://arxiv.org/abs/2307.08621) ![](https://img.shields.io/badge/arXiv-2023.07-red)
- [MiniMax-01: Scaling Foundation Models with Lightning Attention](https://arxiv.org/abs/2501.08313) ![](https://img.shields.io/badge/arXiv-2025.01-red)
- [RWKV: Reinventing RNNs for the Transformer Era](https://arxiv.org/abs/2305.13048) ![](https://img.shields.io/badge/arXiv-2023.05-red)
- [Eagle and Finch: RWKV with Matrix-Valued States and Dynamic Recurrence](https://arxiv.org/abs/2404.05892) ![](https://img.shields.io/badge/arXiv-2024.04-red)
- [Gated Linear Attention Transformers with Hardware-Efficient Training](https://arxiv.org/abs/2312.06635) ![](https://img.shields.io/badge/arXiv-2023.12-red)
- [HGRN2: Gated Linear RNNs with State Expansion](https://arxiv.org/abs/2404.07904) ![](https://img.shields.io/badge/arXiv-2024.04-red)
- [Gated Delta Networks: Improving Mamba2 with Delta Rule](https://proceedings.iclr.cc/paper_files/paper/2025/hash/4904fad153f6434a7bcf04465d4be2cc-Abstract-Conference.html) ![](https://img.shields.io/badge/arXiv-2024.12-red) ![](https://img.shields.io/badge/ICLR-2025-yellow)
- [RWKV-7 "Goose" with Expressive Dynamic State Evolution](https://arxiv.org/abs/2503.14456) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [Kimi Linear: An Expressive, Efficient Attention Architecture](https://arxiv.org/abs/2510.26692) ![](https://img.shields.io/badge/arXiv-2025.10-red)
- [Gated DeltaNet-2: Decoupling Erase and Write in Linear Attention](https://arxiv.org/abs/2605.22791) ![](https://img.shields.io/badge/arXiv-2026.05-red)
- [Erase-then-Delta Attention: Decoupling Erase and Write Addresses in Delta-Rule Linear Attention](https://arxiv.org/abs/2606.26560) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [Learning to (Learn at Test Time): RNNs with Expressive Hidden States](https://arxiv.org/abs/2407.04620) ![](https://img.shields.io/badge/arXiv-2024.07-red)
- [Test-time regression: a unifying framework for designing sequence models with associative memory](https://www.jmlr.org/papers/v27/25-0903.html) ![](https://img.shields.io/badge/JMLR-2026-yellow)

<a id="capacity-expansion"></a>

### 容量扩展

该类别通过增加可寻址状态资源缓解固定状态容量瓶颈，包括仅激活选定分区的稀疏状态扩展和选择特定状态行进行 Delta 式操作的稀疏 Delta 记忆。

- [Scaling Linear Attention with Sparse State Expansion](https://arxiv.org/abs/2507.16577) ![](https://img.shields.io/badge/arXiv-2025.07-red)
- [Sparse Delta Memory: Scaling the State of Linear RNNs through Sparsity](https://arxiv.org/abs/2607.07386) ![](https://img.shields.io/badge/arXiv-2026.07-red)

<a id="temporal-expansion"></a>

### 时间扩展

该类别扩展线性注意力能够表示的时间范围，涵盖多时间尺度上的对数线性分组、根据序列内容自适应调整的动态分段，以及为不同关联或时间尺度维护多个循环状态的 token 级多头状态。

- [Log-Linear Attention](https://arxiv.org/abs/2506.04761) ![](https://img.shields.io/badge/arXiv-2025.06-red)
- [Dynamic Linear Attention](https://arxiv.org/abs/2606.10650) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [MHLA: Restoring Expressivity of Linear Attention via Token-Level Multi-Head](https://arxiv.org/abs/2601.07832) ![](https://img.shields.io/badge/arXiv-2026.01-red)

<a id="auxiliary-coordination-and-routing"></a>

### 辅助协调与路由

该类别在基础递归状态之外增加协调机制，包括在独立特征头之间恢复竞争或通信的特征头协调和跨层路由或复用循环状态的跨深度路由。

- [Softmax Linear Attention: Reclaiming Global Competition](https://arxiv.org/abs/2602.01744) ![](https://img.shields.io/badge/arXiv-2026.02-red)
- [Linear Attention Architectures: Mechanisms, Trade-offs, and Cross-Layer Routing](https://arxiv.org/abs/2607.07953) ![](https://img.shields.io/badge/arXiv-2026.07-red)

<a id="state-space-models"></a>

## 状态空间模型


状态空间模型通过结构化状态动力学压缩历史，其发展从稳定的时不变传播走向输入条件控制，以及更明确的读写接口和状态几何。

<a id="state-dynamics-and-selective-control"></a>

### 状态动力学与选择性控制

该类别关注状态沿序列传播时的动力学设计，从各序列位置应用共享稳定转移的结构化时不变动力学，到由当前输入调节状态控制的输入条件选择性动力学。

- [HiPPO: Recurrent Memory with Optimal Polynomial Projections](https://arxiv.org/abs/2008.07669) ![](https://img.shields.io/badge/arXiv-2020.08-red) ![](https://img.shields.io/badge/NeurIPS-2020-yellow)
- [Combining Recurrent, Convolutional, and Continuous-time Models with Linear State-Space Layers](https://arxiv.org/abs/2110.13985) ![](https://img.shields.io/badge/arXiv-2021.10-red) ![](https://img.shields.io/badge/NeurIPS-2021-yellow)
- [Efficiently Modeling Long Sequences with Structured State Spaces](https://openreview.net/forum?id=uYLFoz1vlAC) ![](https://img.shields.io/badge/arXiv-2021.11-red) ![](https://img.shields.io/badge/ICLR-2022-yellow)
- [On the Parameterization and Initialization of Diagonal State Space Models](https://arxiv.org/abs/2206.11893) ![](https://img.shields.io/badge/arXiv-2022.06-red)
- [Simplified State Space Layers for Sequence Modeling](https://arxiv.org/abs/2208.04933) ![](https://img.shields.io/badge/arXiv-2022.08-red)
- [Mamba: Linear-Time Sequence Modeling with Selective State Spaces](https://openreview.net/forum?id=tEYskw1VY2) ![](https://img.shields.io/badge/arXiv-2023.12-red) ![](https://img.shields.io/badge/COLM-2024-yellow)
- [Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality](https://proceedings.mlr.press/v235/dao24a.html) ![](https://img.shields.io/badge/arXiv-2024.05-red) ![](https://img.shields.io/badge/ICML-2024-yellow)

<a id="state-organization-and-read-write-refinements"></a>

### 状态组织与读写细化

该类别细化状态组织与访问方式，包括显式分离或控制读写路径的读写接口和使用分块、滤波或受约束几何来组织状态和更新的状态组织与更新几何。

- [Hungry Hungry Hippos: Towards Language Modeling with State Space Models](https://arxiv.org/abs/2212.14052) ![](https://img.shields.io/badge/arXiv-2022.12-red)
- [Zoology: Measuring and Improving Recall in Efficient Language Models](https://arxiv.org/html/2312.04927v1) ![](https://img.shields.io/badge/arXiv-2023.12-red) ![](https://img.shields.io/badge/ICLR-2024-yellow)
- [Mamba-3: Improved Sequence Modeling using State Space Principles](https://arxiv.org/abs/2603.15569) ![](https://img.shields.io/badge/arXiv-2026.03-red)
- [MIMOMamba: From Scalar Duality to Matrix-Valued Attention](https://openreview.net/forum?id=UmQ07sj13y) ![](https://img.shields.io/badge/ICML-2026-yellow)
- [Graph Signal Processing Meets Mamba2: Adaptive Filter Bank via Delta Modulation](https://openreview.net/forum?id=w0XhHcXfKv) ![](https://img.shields.io/badge/ICLR-2026-yellow)
- [MuonSSM: Orthogonalizing State Space Models for Sequence Modeling](https://arxiv.org/abs/2606.30461) ![](https://img.shields.io/badge/arXiv-2026.06-red) ![](https://img.shields.io/badge/ICML-2026-yellow)
- [The Illusion of State in State-Space Models](https://arxiv.org/html/2404.08819v3) ![](https://img.shields.io/badge/arXiv-2024.04-red) ![](https://img.shields.io/badge/ICML-2024-yellow)

**分析性参考。** *The Illusion of State in State-Space Models* 分析了特定状态转移类别与数值假设下的状态跟踪局限。本文将其作为理解相关限制的参考，不为其分配机制修改标签。

<a id="hybrid-architecture"></a>

## 混合架构


混合架构不是独立的记忆算子，而是在层、头、分支或 token 等不同结构粒度组合 Softmax、Sparse、Linear Attention 与 SSM。

<a id="layer-wise-hybrid"></a>

### 层级混合

该类别沿网络深度分配异构序列混合器，涵盖预定义分配、直接诊断选择、基于对齐的选择和迭代蒸馏引导选择。

- [Griffin: Mixing Gated Linear Recurrences with Local Attention for Efficient Language Models](https://arxiv.org/abs/2402.19427) ![](https://img.shields.io/badge/arXiv-2024.02-red)
- [Jamba: A Hybrid Transformer-Mamba Language Model](https://arxiv.org/abs/2403.19887) ![](https://img.shields.io/badge/arXiv-2024.03-red)
- [Samba: Simple Hybrid State Space Models for Efficient Unlimited Context Language Modeling](https://arxiv.org/abs/2406.07522) ![](https://img.shields.io/badge/arXiv-2024.06-red)
- [Zamba: A Compact 7B SSM Hybrid Model](https://arxiv.org/abs/2405.16712) ![](https://img.shields.io/badge/arXiv-2024.05-red)
- [The Zamba2 Suite: Technical Report](https://arxiv.org/abs/2411.15242) ![](https://img.shields.io/badge/arXiv-2024.11-red)
- [HySparse: A Hybrid Sparse Attention Architecture with Oracle Token Selection and KV Cache Sharing](https://arxiv.org/abs/2602.03560) ![](https://img.shields.io/badge/arXiv-2026.02-red)
- [HySparse2: Hybrid Sparse Attention with Two-Level KV Sharing](https://arxiv.org/abs/2609.26368) ![](https://img.shields.io/badge/arXiv-2026.09-red)
- [LightTransfer: Your Long-Context LLM is Secretly a Hybrid Model with Effortless Adaptation](https://arxiv.org/abs/2410.13846) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Priming: Hybrid State Space Models From Pre-trained Transformers](https://arxiv.org/abs/2605.08301) ![](https://img.shields.io/badge/arXiv-2026.05-red)
- [Jet-Nemotron: Efficient Language Model with Post Neural Architecture Search](https://arxiv.org/abs/2508.15884) ![](https://img.shields.io/badge/arXiv-2025.08-red) ![](https://img.shields.io/badge/NeurIPS-2025-yellow)
- [Hybrid Linear Attention Done Right: Efficient Distillation and Effective Architectures for Extremely Long Contexts](https://arxiv.org/abs/2601.22156) ![](https://img.shields.io/badge/arXiv-2026.01-red)
- [Distilling to Hybrid Attention Models via KL-Guided Layer Selection](https://arxiv.org/abs/2512.20569) ![](https://img.shields.io/badge/arXiv-2025.12-red)

<a id="head-wise-hybrid"></a>

### 头级混合

该类别在同一层内按注意力头或通道组组合不同机制，涵盖固定头分配、功能感知头选择、输入自适应头路由和深度自适应头分配。

- [Hymba: A Hybrid-head Architecture for Small Language Models](https://proceedings.iclr.cc/paper_files/paper/2025/hash/f32def07618040e540e0a6182e290562-Abstract-Conference.html) ![](https://img.shields.io/badge/arXiv-2024.11-red) ![](https://img.shields.io/badge/ICLR-2025-yellow)
- [Falcon-H1: A Family of Hybrid-Head Language Models Redefining Efficiency and Performance](https://arxiv.org/abs/2507.22448) ![](https://img.shields.io/badge/arXiv-2025.07-red)
- [DuoAttention: Efficient Long-Context LLM Inference with Retrieval and Streaming Heads](https://arxiv.org/abs/2410.10819) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [HydraHead: From Head-Level Functional Heterogeneity to Specialized Attention Hybridization](https://arxiv.org/abs/2606.20097) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [Elastic Attention: Test-time Adaptive Sparsity Ratios for Efficient Transformers](https://arxiv.org/abs/2601.17367) ![](https://img.shields.io/badge/arXiv-2026.01-red)
- [Modern Transformers Are Implicit Hybrids: From Functional Differentiation to Principled Hybrid Architecture Design](https://arxiv.org/abs/2609.02986) ![](https://img.shields.io/badge/arXiv-2026.09-red)

<a id="branch-wise-hybrid"></a>

### 分支级混合

该类别在同一 block 内维护多条记忆路径，包括在输出投影前合并各分支读出的读出级分支融合和通过相加、门控或投影合并独立输出的输出级分支融合。Memorizing Transformer 作为边界案例保留：其外部 kNN 数据库不属于核心分类所强调的模型内部记忆，但门控读出接口可与内部记忆的分支融合直接比较。

- [Block-Recurrent Transformers](https://proceedings.neurips.cc/paper_files/paper/2022/hash/d6e0bbb9fc3f4c10950052ec2359355c-Abstract-Conference.html) ![](https://img.shields.io/badge/arXiv-2022.03-red) ![](https://img.shields.io/badge/NeurIPS-2022-yellow)
- [Memorizing Transformers](https://arxiv.org/abs/2203.08913) ![](https://img.shields.io/badge/arXiv-2022.03-red)
- [Leave No Context Behind: Efficient Infinite Context Transformers with Infini-attention](https://arxiv.org/abs/2404.07143) ![](https://img.shields.io/badge/arXiv-2024.04-red)
- [DART: Decoded Attention over Recurrent States for Efficient Long-Context Sequence Modeling](https://arxiv.org/abs/2608.02032) ![](https://img.shields.io/badge/arXiv-2026.08-red)
- [Titans: Learning to Memorize at Test Time](https://arxiv.org/abs/2501.00663) ![](https://img.shields.io/badge/arXiv-2025.01-red)

<a id="token-wise-hybrid"></a>

### token 级混合

该类别为不同 token 或分块选择记忆路径，涵盖基于位置的时间边界分配、使用内容相关性得分的内容得分分配，以及通过训练路由器实现的学习式操作分配。

- [LoLCATs: On Low-Rank Linearizing of Large Language Models](https://arxiv.org/abs/2410.10254) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Native Hybrid Attention for Efficient Sequence Modeling](https://aclanthology.org/2026.acl-long.176/) ![](https://img.shields.io/badge/arXiv-2025.10-red) ![](https://img.shields.io/badge/ACL-2026-yellow)
- [LoLA: Low-Rank Linear Attention With Sparse Caching](https://arxiv.org/abs/2505.23666) ![](https://img.shields.io/badge/arXiv-2025.05-red)
- [STILL: Selecting Tokens for Intra-Layer Hybrid Attention to Linearize LLMs](https://arxiv.org/abs/2602.02180) ![](https://img.shields.io/badge/arXiv-2026.02-red)
- [Neural Attention Search Linear: Towards Adaptive Token-Level Hybrid Attention Models](https://arxiv.org/abs/2602.03681) ![](https://img.shields.io/badge/arXiv-2026.02-red)


<a id="contributing"></a>

## 参与贡献


本仓库旨在成为由社区共同维护、持续更新的动态综述。

新增或修订论文条目时，请提供：

- **标题与链接**
- **发表场所与年份**
- **研究路线：** Softmax 注意力、稀疏注意力、线性注意力、状态空间模型或混合架构
- **子类别：** 从相关综述章节列出的子类别中选择最合适的一项
- **五维映射：** 记忆表示、记忆更新、访问、读出和/或集成
- **归类依据：** 根据论文中的具体证据，简要说明建议的归属与五维映射

如果发现重要工作缺失、元数据错误、链接失效，或对已有分类存在异议，请提交 Issue 或 Pull Request，并附上建议修改及相关证据。修改共有内容时，请保持 `README.md` 与 `README-ZH.md` 一致。


<a id="citation"></a>

## 引用


```bibtex
@misc{tan2026evolution,
  title         = {The Evolution of Attention in Large Language Models: Mechanisms, Trade-offs, and Emerging Trends},
  author        = {Zhentao Tan and Jingyi Shen and Yanbo Li and Yao Liu and Yue Wu and Jieping Ye},
  year          = {2026},
  eprint        = {2609.39661},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2609.39661}
}
```
