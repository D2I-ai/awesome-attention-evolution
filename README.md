<div align="center">

<h1>The Evolution of Attention in Large Language Models: Mechanisms, Trade-offs, and Emerging Trends</h1>

<p><strong>Zhentao Tan, Jingyi Shen, Yanbo Li, Yao Liu, Yue Wu, Jieping Ye</strong><br>
Alibaba Token Hub, Alibaba Group</p>

[![Paper](https://img.shields.io/badge/Paper-arXiv-b31b1b.svg)](https://arxiv.org/pdf/2609.39661)
[![Awesome](https://awesome.re/badge.svg)](#survey-index)
[![128 Papers](https://img.shields.io/badge/Papers-128-yellow.svg)](#survey-index)
[![Contributions Welcome](https://img.shields.io/badge/Contribution-Welcome-brightgreen.svg)](#contributing)
[![Citation](https://img.shields.io/badge/Citation-BibTeX-blue.svg)](#citation)

</div>

<div align="center">

**English** | [中文版](./README-ZH.md)

[Full Text](https://arxiv.org/abs/2609.39661)

A living survey and structured literature map of efficient sequence architectures, explaining how they have evolved from a **memory-centric perspective**.

</div>

## Overview


This repository accompanies the survey “[The Evolution of Attention in Large Language Models: Mechanisms, Trade-offs, and Emerging Trends](https://arxiv.org/abs/2609.39661).”

The survey examines efficient sequence architectures—from Softmax and Sparse Attention to Linear Attention, State Space Models, and Hybrid designs—through a unifying concept: **model-internal contextual memory**. As new architectures continue to emerge rapidly, it asks a common question: beneath their diverse designs, are they all solving the same problem—how to manage their own contextual memory?

To connect research directions that are often studied in isolation, the survey introduces a five-dimensional analytical lens:

- **Memory Representation** — what form the context is stored in
- **Memory Update** — how new information is written
- **Access** — how stored memory is selected
- **Readout** — how it is retrieved
- **Integration** — how it is combined back into computation

<div align="center">

![Five-dimensional memory-centric framework](./assets/attention-primer.png)

</div>

Drawing on 59 release-level records across 14 major model lineages, together with a structured comparison of 11 high-performing open-weight models, the analysis shows that memory processing is increasingly coordinated not only across time, but also across network depth.

The resulting framework provides a common language for comparing today's architectures and points toward future designs built on stateful, multidimensional, and selectively routed memory.

## Core Framework


Different sequence architectures make preceding context available through visibly different operators. Softmax Attention retains separately addressable units, Sparse Attention limits which units are examined, and recurrent mechanisms continually compress history into one or more states. The survey compares them functionally as systems that maintain and use **contextual memory**: model-internal, input-dependent information available while processing the current sequence. This includes token KV entries, compressed tokens or chunks, memory slots, associative matrices, structured recurrent states, and heterogeneous combinations, but excludes pretrained parameters and external retrieval corpora.

Let $t$ be the current sequence position, $x_t$ the input to the memory mechanism, and $q_t$ the representation that queries contextual memory. A memory schema $\rho$ specifies the representation type, organization, granularity, capacity, and persistence.

**1. Memory Representation.** The maintained memory $\mathcal{M}_t$ must belong to the state space admitted by its schema:

<div align="center">

$$
\mathcal{M}_t \in \mathfrak{M}_{\rho}.
$$

</div>

The schema may describe a growing list of token KVs, bounded summary slots, an associative matrix, a structured recurrent state, or several heterogeneous paths. Representation therefore asks both how much history remains covered and whether individual historical units remain separately addressable.

**2. Memory Update.** Let $\mathcal{M}_t^{-}$ and $\mathcal{M}_t^{+}$ denote memory immediately before and after processing the current input. Update is defined abstractly as:

<div align="center">

$$
\mathcal{M}_t^{+}=\mathrm{Update}_{\rho}\!\left(\mathcal{M}_t^{-},x_t\right).
$$

</div>

This operator covers append-only KV caches, compression into bounded slots, recurrent state transitions, decay, Delta corrections, and controlled erase-write rules. It specifies how memory changes, not which parts a query will use.

**3. Access.** Let $\widehat{\mathcal{M}}_t$ be the memory view exposed to the read path; depending on the mechanism, it may be the state before or after the current update. Access constructs the eligible view $\mathcal{C}_t$:

<div align="center">

$$
\mathcal{C}_t=\mathrm{Access}_{\rho}\!\left(q_t,\widehat{\mathcal{M}}_t\right).
$$

</div>

The result may contain all causal token memories, a local or routed subset of tokens or blocks, one or more recurrent-state interfaces, and any routing metadata needed by the next step.

**4. Readout.** Readout determines how the query extracts contextual information from the eligible view:

<div align="center">

$$
r_t=\mathrm{Readout}_{\rho}\!\left(q_t,\mathcal{C}_t\right).
$$

</div>

Examples include normalized query-key aggregation, associative-state contraction, and structured state projection. Thus, Access answers **what may be read**, whereas Readout answers **how eligible information is weighted, decoded, or aggregated**.

**5. Integration.** Integration transforms one or more completed readouts into the memory module output:

<div align="center">

$$
o_t=\mathrm{Integration}_{\rho}\!\left(r_t;x_t\right).
$$

</div>

Here $r_t$ may be one vector or a collection of head-, branch-, or path-level readouts. Integration includes output projection, concatenation, summation, gating, conditional routing, and fusion across heterogeneous memory paths.

These equations define functional roles rather than a universal computational graph. One operation may implement several roles at once: changing Representation can alter Access, an input-conditioned transition can jointly change Representation and Update, and a hierarchical sparse index can affect both address representation and candidate selection.

<a id="classification-logic-and-family-mapping"></a>

## Classification Logic and Family Mapping


The survey taxonomy and the five-dimensional framework serve different purposes. The five research lines follow their principal technical questions and historical development; they are not mechanically derived from the five dimensions. Each work is assigned a principal research line according to its central contribution and lineage. A small number of cross-cutting methods are also indexed in a secondary table when they substantively contribute to another design direction; consequently, the list contains 131 classification records for 128 unique works. Dimensional tags provide a multi-label description of the memory functions each method modifies, allowing methods from different research lines to be compared when they intervene in the same functional role.

**P** marks a dimension commonly treated as a primary design target, while **S** marks a dimension usually inherited from the base mechanism or modified only in particular subfamilies. The mapping is interpretive rather than exhaustive: all five dimensions remain applicable to every research line.

| Research line | Memory substrate | Representation | Update | Access | Readout | Integration |
|---|---|:---:|:---:|:---:|:---:|:---:|
| Softmax Attention | Explicit memory units | **P** | **P** | **S** | **S** | **S** |
| Sparse Attention | Explicit token/block KV | **S** | **S** | **P** | **S** | **S** |
| Linear Attention | Associative recurrent state | **P** | **P** | **S** | **S** | **S** |
| State Space Models | Structured recurrent state | **P** | **P** | **S** | **S** | **S** |
| Hybrid Architecture | Heterogeneous memory substrates | **P** | **S** | **S** | **S** | **S** |

Softmax Attention primarily reduces redundancy or reorganizes explicit memory through Representation and Update. Sparse Attention usually preserves token- or block-level content memory and a Softmax Readout, while making candidate construction and access-budget allocation its central Access problem. Linear Attention and State Space Models focus on how recurrent states are represented and updated; Access, Readout, and Integration become more explicit in routed, gated, or multi-state variants.

Hybrid Architecture requires a different interpretation because it composes heterogeneous mechanisms or access regimes rather than introducing one additional memory operator. Representation is therefore its primary family-level axis, while its effects on Update, Access, Readout, and Integration depend on whether composition occurs across layers, heads, branches, or tokens.

<a id="architecture-landscape-and-future-directions"></a>

## Architecture Landscape and Future Directions


Publicly documented LLM attention is **diversifying rather than converging**. Across 59 release-level records from 14 major model lineages, 58 are classifiable: 32 use one principal family and 26 are Hybrid, with Hybrid records rising from 0/6 in 2022–2023 to 17/27 in 2026. Most Hybrid designs still use predetermined layer-wise schedules, while only five records explicitly reuse KV representations, indexes, or Top-k decisions across layers. The frozen frontier snapshot below reinforces this coexistence: the 11 selected high-performing open-weight model endpoints span full GQA, sparse attention, latent compression, and recurrent–explicit hybrids, yet every architecture retains an explicit token-retrieval path. These observations describe adoption patterns and do not attribute model quality to any single mechanism.

**Attention structures of selected high-performing open-weight model endpoints.** The snapshot was captured on September 22, 2026; model size reports total parameters, with disclosed active parameters in parentheses.

| Model endpoint | First Release | Model size | Attention structure |
|---|:---:|:---:|---|
| MiMo-V2.6-Pro | 2026.09 | 1.02T (42B active) | 60 SWA-GQA layers + 10 global-GQA layers |
| DeepSeek-V4.1-Flash (max) | 2026.09 | 552B (8B active prefill; 16B active decode) | CSA2 with SWA; cross-layer KV, index, and Top-k reuse |
| K2-Horizon-375B-A23B | 2026.09 | 375B (23B active) | Full GQA |
| GLM-5.3-Flash | 2026.08 | 320B (18B active) | 34 KDA layers + 11 compressed-indexer DSA layers |
| GLM-5.3 (max) | 2026.08 | 744B (40B active) | MLA-based DSA with IndexShare/IndexCache |
| Qwen3.8-Flash-Next | 2026.08 | 176B; 125B main (6B active) | 3 GDN layers per 1 QSA layer |
| DeepSeek-V4 Pro 0813 (max) | 2026.08 | 1.6T (49B active) | CSA/HCA layer-wise hybrid |
| Qwen3.8-2.4T-A95B | 2026.08 | 2.4T (95B active) | 3 Gated DeltaNet layers per 1 gated full-GQA layer |
| Qwen3.8-27B (xhigh) | 2026.08 | 27B | 3 Gated DeltaNet layers per 1 gated full-GQA layer |
| Kimi K3 (max) | 2026.07 | 2.8T (104B active) | 3 KDA layers per 1 Gated MLA layer |
| MiniMax-M3 | 2026.06 | 428B (23B active) | 3 dense-attention layers + 57 MSA layers |

### Three-Level Synthesis

Rather than viewing architectural evolution as a succession of operator replacements, the survey interprets it as an expanding redesign of contextual memory—from mechanism-level control, through architecture-level coordination, to future stateful memory systems:

> 🔵 **Mechanism Level — Expanding Control Scopes**
>
> Explicit-memory and state-based methods retain different memory interfaces, but both expand control across an increasingly overlapping set of memory functions.

> 🔵 **Architecture Level — Memory Organization across Depth**
>
> Layer-wise composition distributes complementary memory processing across representational stages, while cross-layer reuse extends the lifetime of selected memory and routing artifacts. Together, they make network depth an emerging dimension of memory organization.

> 🔵 **Forward-Looking Level — Stateful Multidimensional Memory Routing**
>
> The depth-wise perspective combines with temporal scope, substrate type, and representation granularity to motivate coordinated Sparse Write and Sparse Read over persistent contextual memory.

<a id="literature-index"></a>

## Survey Index


- [Softmax Attention](#softmax-attention)
  - [Memory-Representation Efficiency](#memory-representation-efficiency)
  - [Sequence-Representation Compression](#sequence-representation-compression)
  - [Readout and Integration Modulation](#readout-and-integration-modulation)
- [Sparse Attention](#sparse-attention)
  - [Structure-Constrained Sparse Attention](#structure-constrained-sparse-attention)
  - [Self-Routing Sparse Attention](#self-routing-sparse-attention)
  - [Auxiliary-Proxy Sparse Routing](#auxiliary-proxy-sparse-routing)
  - [Temporal and Cross-Layer Reuse](#temporal-and-cross-layer-reuse)
- [Linear Attention](#linear-attention)
  - [Memory Update Rules](#memory-update-rules)
  - [Capacity Expansion](#capacity-expansion)
  - [Temporal Expansion](#temporal-expansion)
  - [Auxiliary Coordination and Routing](#auxiliary-coordination-and-routing)
- [State Space Models](#state-space-models)
  - [State Dynamics and Selective Control](#state-dynamics-and-selective-control)
  - [State Organization and Read-Write Refinements](#state-organization-and-read-write-refinements)
- [Hybrid Architecture](#hybrid-architecture)
  - [Layer-wise Hybrid](#layer-wise-hybrid)
  - [Head-wise Hybrid](#head-wise-hybrid)
  - [Branch-wise Hybrid](#branch-wise-hybrid)
  - [Token-wise Hybrid](#token-wise-hybrid)


<a id="softmax-attention"></a>

## Softmax Attention


Softmax Attention preserves explicitly enumerable memory units and normalized Softmax readout. Its main design directions reduce per-token representation cost, compress long histories, or modify how attention scores and head outputs are integrated.

<a id="memory-representation-efficiency"></a>

### Memory-Representation Efficiency

This category reduces the KV representation cost of each token while retaining exact Softmax readout, spanning the head-sharing continuum from MHA through GQA to MQA, channel compression into lower-dimensional latent KV, and cross-layer KV sharing.

- [Attention Is All You Need](https://proceedings.neurips.cc/paper/7181-attention-is-all-you-need) ![](https://img.shields.io/badge/arXiv-2017.06-red) ![](https://img.shields.io/badge/NeurIPS-2017-yellow)
- [Fast Transformer Decoding: One Write-Head is All You Need](https://arxiv.org/abs/1911.02150) ![](https://img.shields.io/badge/arXiv-2019.11-red)
- [GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://aclanthology.org/2023.emnlp-main.298/) ![](https://img.shields.io/badge/arXiv-2023.05-red) ![](https://img.shields.io/badge/EMNLP-2023-yellow)
- [DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model](https://arxiv.org/abs/2405.04434) ![](https://img.shields.io/badge/arXiv-2024.05-red)
- [TransMLA: Multi-Head Latent Attention Is All You Need](https://arxiv.org/abs/2502.07864) ![](https://img.shields.io/badge/arXiv-2025.02-red)
- [Reducing Transformer Key-Value Cache Size with Cross-Layer Attention](https://arxiv.org/abs/2405.12981) ![](https://img.shields.io/badge/arXiv-2024.05-red)

<a id="sequence-representation-compression"></a>

### Sequence-Representation Compression

This category compresses history as sequence length grows, through fixed-capacity recurrent memory that consolidates history into bounded states and dynamic-resolution memory that preserves recent detail with progressively coarser older summaries.

- [Transformer-XL: Attentive Language Models Beyond a Fixed-Length Context](https://arxiv.org/abs/1901.02860) ![](https://img.shields.io/badge/arXiv-2019.01-red)
- [Compressive Transformers for Long-Range Sequence Modelling](https://arxiv.org/abs/1911.05507) ![](https://img.shields.io/badge/arXiv-2019.11-red)
- [Recurrent Memory Transformer](https://arxiv.org/abs/2207.06881) ![](https://img.shields.io/badge/arXiv-2022.07-red)
- [TransformerFAM: Feedback attention is working memory](https://arxiv.org/abs/2404.09173) ![](https://img.shields.io/badge/arXiv-2024.04-red)
- [Trellis: Learning to Compress Key-Value Memory in Attention Models](https://arxiv.org/abs/2512.23852) ![](https://img.shields.io/badge/arXiv-2025.12-red)
- [Lattice: Learning to Compress the Cache in the Attention](https://research.google/pubs/lattice-learning-to-compress-the-cache-in-the-attention/) ![](https://img.shields.io/badge/Google_Research-2025-yellow)
- [Kwai Summary Attention Technical Report](https://arxiv.org/abs/2604.24432) ![](https://img.shields.io/badge/arXiv-2026.04-red)
- [DeepSeek-V4: Towards Highly Efficient Million-Token Context Intelligence](https://arxiv.org/abs/2606.19348) ![](https://img.shields.io/badge/arXiv-2026.06-red)

<a id="readout-and-integration-modulation"></a>

### Readout and Integration Modulation

This category leaves the underlying memory set intact and changes attention readout or multi-head integration, including score and attention-map modulation before value aggregation and head routing with output gating during integration.

- [Talking-Heads Attention](https://arxiv.org/abs/2003.02436) ![](https://img.shields.io/badge/arXiv-2020.03-red)
- [Improving Transformers with Dynamically Composable Multi-Head Attention](https://arxiv.org/abs/2405.08553) ![](https://img.shields.io/badge/arXiv-2024.05-red)
- [Differential Transformer](https://arxiv.org/abs/2410.05258) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Forgetting Transformer: Softmax Attention with a Forget Gate](https://arxiv.org/abs/2503.02130) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [MoH: Multi-Head Attention as Mixture-of-Head Attention](https://arxiv.org/abs/2410.11842) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Gated Attention for Large Language Models: Non-linearity, Sparsity, and Attention-Sink-Free](https://arxiv.org/abs/2505.06708) ![](https://img.shields.io/badge/arXiv-2025.05-red)

<a id="sparse-attention"></a>

## Sparse Attention


Sparse Attention retains fine-grained KV candidates while restricting the support read by each query. The taxonomy distinguishes where sparse supports come from, at what granularity they are formed, and whether routing results are reused.

<a id="structure-constrained-sparse-attention"></a>

### Structure-Constrained Sparse Attention

This category restricts accessible KV pairs with stable structural patterns, including architecture-prescribed topologies defined before training and post-hoc discovered rules derived from regularities in trained dense models.

- [Generating Long Sequences with Sparse Transformers](https://arxiv.org/abs/1904.10509) ![](https://img.shields.io/badge/arXiv-2019.04-red)
- [Longformer: The Long-Document Transformer](https://arxiv.org/abs/2004.05150) ![](https://img.shields.io/badge/arXiv-2020.04-red)
- [Big Bird: Transformers for Longer Sequences](https://proceedings.neurips.cc/paper/2020/hash/c8512d142a2d849725f31a9a7a361ab9-Abstract.html) ![](https://img.shields.io/badge/arXiv-2020.07-red) ![](https://img.shields.io/badge/NeurIPS-2020-yellow)
- [LongNet: Scaling Transformers to 1,000,000,000 Tokens](https://arxiv.org/abs/2307.02486) ![](https://img.shields.io/badge/arXiv-2023.07-red)
- [PowerAttention: Exponentially Scaling of Receptive Fields for Effective Sparse Attention](https://arxiv.org/abs/2503.03588) ![](https://img.shields.io/badge/arXiv-2025.03-red)
- [Efficient Streaming Language Models with Attention Sinks](https://arxiv.org/abs/2309.17453) ![](https://img.shields.io/badge/arXiv-2023.09-red)
- [LM-Infinite: Zero-Shot Extreme Length Generalization for Large Language Models](https://arxiv.org/abs/2308.16137) ![](https://img.shields.io/badge/arXiv-2023.08-red)
- [MInference 1.0: Accelerating Pre-filling for Long-Context LLMs via Dynamic Sparse Attention](https://arxiv.org/abs/2407.02490) ![](https://img.shields.io/badge/arXiv-2024.07-red)

<a id="self-routing-sparse-attention"></a>

### Self-Routing Sparse Attention

This category derives sparse candidates from the query and context themselves, through dynamic token grouping, block aggregation and selection, and summary-token routing.

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

### Auxiliary-Proxy Sparse Routing

This category uses a lightweight auxiliary indexer to estimate candidate supports, covering token-granularity proxy routing that ranks individual entries and block or compressed-entry proxy routing that ranks aggregated representations.

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

### Temporal and Cross-Layer Reuse

This category amortizes indexing cost by reusing previously computed sparse supports, including temporal support reuse across nearby queries or decoding steps and cross-layer index reuse across network depth.

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

## Linear Attention


Linear Attention replaces an enumerable token cache with one or more recurrent associative states. The canonical formulation uses context-length-independent state, while later variants expand, partition, or organize multiple states to increase capacity and temporal coverage. Its evolution centers on controlled state editing, capacity growth, broader temporal coverage, and auxiliary coordination or routing.

<a id="memory-update-rules"></a>

### Memory Update Rules

This category studies how associative state is retained, erased, and written, spanning additive and corrective updates (including Delta-style corrections), retention and gated state editing, and loss-based online optimization.

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

### Capacity Expansion

This category addresses fixed-state capacity limits by increasing addressable state resources, through sparse state expansion that activates selected partitions and sparse Delta memory that targets particular state rows.

- [Scaling Linear Attention with Sparse State Expansion](https://arxiv.org/abs/2507.16577) ![](https://img.shields.io/badge/arXiv-2025.07-red)
- [Sparse Delta Memory: Scaling the State of Linear RNNs through Sparsity](https://arxiv.org/abs/2607.07386) ![](https://img.shields.io/badge/arXiv-2026.07-red)

<a id="temporal-expansion"></a>

### Temporal Expansion

This category broadens the temporal range represented by linear attention, through log-linear grouping at multiple timescales, dynamic segmentation adapted to sequence content, and token-level multi-head state for distinct associations.

- [Log-Linear Attention](https://arxiv.org/abs/2506.04761) ![](https://img.shields.io/badge/arXiv-2025.06-red)
- [Dynamic Linear Attention](https://arxiv.org/abs/2606.10650) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [MHLA: Restoring Expressivity of Linear Attention via Token-Level Multi-Head](https://arxiv.org/abs/2601.07832) ![](https://img.shields.io/badge/arXiv-2026.01-red)

<a id="auxiliary-coordination-and-routing"></a>

### Auxiliary Coordination and Routing

This category adds coordination around the base recurrence, including feature-head coordination that restores competition across independent heads and cross-depth routing that reuses recurrent states across layers.

- [Softmax Linear Attention: Reclaiming Global Competition](https://arxiv.org/abs/2602.01744) ![](https://img.shields.io/badge/arXiv-2026.02-red)
- [Linear Attention Architectures: Mechanisms, Trade-offs, and Cross-Layer Routing](https://arxiv.org/abs/2607.07953) ![](https://img.shields.io/badge/arXiv-2026.07-red)

<a id="state-space-models"></a>

## State Space Models


State Space Models compress history through structured state dynamics. The progression runs from stable time-invariant propagation to input-conditioned control and increasingly explicit read-write interfaces and state geometry.

<a id="state-dynamics-and-selective-control"></a>

### State Dynamics and Selective Control

This category concerns how state propagates through a sequence, from structured time-invariant dynamics with stable learned transitions to input-conditioned selective dynamics where the current input modulates state control.

- [HiPPO: Recurrent Memory with Optimal Polynomial Projections](https://arxiv.org/abs/2008.07669) ![](https://img.shields.io/badge/arXiv-2020.08-red) ![](https://img.shields.io/badge/NeurIPS-2020-yellow)
- [Combining Recurrent, Convolutional, and Continuous-time Models with Linear State-Space Layers](https://arxiv.org/abs/2110.13985) ![](https://img.shields.io/badge/arXiv-2021.10-red) ![](https://img.shields.io/badge/NeurIPS-2021-yellow)
- [Efficiently Modeling Long Sequences with Structured State Spaces](https://openreview.net/forum?id=uYLFoz1vlAC) ![](https://img.shields.io/badge/arXiv-2021.11-red) ![](https://img.shields.io/badge/ICLR-2022-yellow)
- [On the Parameterization and Initialization of Diagonal State Space Models](https://arxiv.org/abs/2206.11893) ![](https://img.shields.io/badge/arXiv-2022.06-red)
- [Simplified State Space Layers for Sequence Modeling](https://arxiv.org/abs/2208.04933) ![](https://img.shields.io/badge/arXiv-2022.08-red)
- [Mamba: Linear-Time Sequence Modeling with Selective State Spaces](https://openreview.net/forum?id=tEYskw1VY2) ![](https://img.shields.io/badge/arXiv-2023.12-red) ![](https://img.shields.io/badge/COLM-2024-yellow)
- [Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality](https://proceedings.mlr.press/v235/dao24a.html) ![](https://img.shields.io/badge/arXiv-2024.05-red) ![](https://img.shields.io/badge/ICML-2024-yellow)

<a id="state-organization-and-read-write-refinements"></a>

### State Organization and Read-Write Refinements

This category refines state organization and access, including explicit read-write interfaces that separate state-reading and state-writing paths and structured state organization with constrained update geometry.

- [Hungry Hungry Hippos: Towards Language Modeling with State Space Models](https://arxiv.org/abs/2212.14052) ![](https://img.shields.io/badge/arXiv-2022.12-red)
- [Zoology: Measuring and Improving Recall in Efficient Language Models](https://arxiv.org/html/2312.04927v1) ![](https://img.shields.io/badge/arXiv-2023.12-red) ![](https://img.shields.io/badge/ICLR-2024-yellow)
- [Mamba-3: Improved Sequence Modeling using State Space Principles](https://arxiv.org/abs/2603.15569) ![](https://img.shields.io/badge/arXiv-2026.03-red)
- [MIMOMamba: From Scalar Duality to Matrix-Valued Attention](https://openreview.net/forum?id=UmQ07sj13y) ![](https://img.shields.io/badge/ICML-2026-yellow)
- [Graph Signal Processing Meets Mamba2: Adaptive Filter Bank via Delta Modulation](https://openreview.net/forum?id=w0XhHcXfKv) ![](https://img.shields.io/badge/ICLR-2026-yellow)
- [MuonSSM: Orthogonalizing State Space Models for Sequence Modeling](https://arxiv.org/abs/2606.30461) ![](https://img.shields.io/badge/arXiv-2026.06-red) ![](https://img.shields.io/badge/ICML-2026-yellow)
- [The Illusion of State in State-Space Models](https://arxiv.org/html/2404.08819v3) ![](https://img.shields.io/badge/arXiv-2024.04-red) ![](https://img.shields.io/badge/ICML-2024-yellow)

**Analytical reference.** *The Illusion of State in State-Space Models* examines state-tracking limitations under specified transition and numerical assumptions. It is included for context and is not assigned a mechanism-modification tag.

<a id="hybrid-architecture"></a>

## Hybrid Architecture


Hybrid Architecture is not a separate memory operator; it composes Softmax, Sparse, Linear Attention, and SSM mechanisms at different structural granularities, including layers, heads, branches, and tokens.

<a id="layer-wise-hybrid"></a>

### Layer-wise Hybrid

This category assigns heterogeneous sequence mixers across network depth, through predefined allocation, direct diagnosis selection, alignment-based selection, and iterative distillation-guided selection.

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

### Head-wise Hybrid

This category combines mechanisms across heads or channel groups within one layer, through fixed head allocation, function-aware head selection, input-adaptive head routing, and depth-adaptive head allocation.

- [Hymba: A Hybrid-head Architecture for Small Language Models](https://proceedings.iclr.cc/paper_files/paper/2025/hash/f32def07618040e540e0a6182e290562-Abstract-Conference.html) ![](https://img.shields.io/badge/arXiv-2024.11-red) ![](https://img.shields.io/badge/ICLR-2025-yellow)
- [Falcon-H1: A Family of Hybrid-Head Language Models Redefining Efficiency and Performance](https://arxiv.org/abs/2507.22448) ![](https://img.shields.io/badge/arXiv-2025.07-red)
- [DuoAttention: Efficient Long-Context LLM Inference with Retrieval and Streaming Heads](https://arxiv.org/abs/2410.10819) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [HydraHead: From Head-Level Functional Heterogeneity to Specialized Attention Hybridization](https://arxiv.org/abs/2606.20097) ![](https://img.shields.io/badge/arXiv-2026.06-red)
- [Elastic Attention: Test-time Adaptive Sparsity Ratios for Efficient Transformers](https://arxiv.org/abs/2601.17367) ![](https://img.shields.io/badge/arXiv-2026.01-red)
- [Modern Transformers Are Implicit Hybrids: From Functional Differentiation to Principled Hybrid Architecture Design](https://arxiv.org/abs/2609.02986) ![](https://img.shields.io/badge/arXiv-2026.09-red)

<a id="branch-wise-hybrid"></a>

### Branch-wise Hybrid

This category maintains multiple memory paths inside one block, through readout-level branch fusion that combines readouts before output projection and output-level branch fusion that merges independently computed outputs. Memorizing Transformer is retained as a boundary comparison: its external kNN datastore falls outside the model-internal memory scope of the core taxonomy, but its gated readout interface is directly comparable to internal branch fusion.

- [Block-Recurrent Transformers](https://proceedings.neurips.cc/paper_files/paper/2022/hash/d6e0bbb9fc3f4c10950052ec2359355c-Abstract-Conference.html) ![](https://img.shields.io/badge/arXiv-2022.03-red) ![](https://img.shields.io/badge/NeurIPS-2022-yellow)
- [Memorizing Transformers](https://arxiv.org/abs/2203.08913) ![](https://img.shields.io/badge/arXiv-2022.03-red)
- [Leave No Context Behind: Efficient Infinite Context Transformers with Infini-attention](https://arxiv.org/abs/2404.07143) ![](https://img.shields.io/badge/arXiv-2024.04-red)
- [DART: Decoded Attention over Recurrent States for Efficient Long-Context Sequence Modeling](https://arxiv.org/abs/2608.02032) ![](https://img.shields.io/badge/arXiv-2026.08-red)
- [Titans: Learning to Memorize at Test Time](https://arxiv.org/abs/2501.00663) ![](https://img.shields.io/badge/arXiv-2025.01-red)

<a id="token-wise-hybrid"></a>

### Token-wise Hybrid

This category chooses memory paths for individual tokens or chunks, through temporal-boundary allocation based on position, content-score allocation using relevance scores, and learned operation allocation via trained routers.

- [LoLCATs: On Low-Rank Linearizing of Large Language Models](https://arxiv.org/abs/2410.10254) ![](https://img.shields.io/badge/arXiv-2024.10-red)
- [Native Hybrid Attention for Efficient Sequence Modeling](https://aclanthology.org/2026.acl-long.176/) ![](https://img.shields.io/badge/arXiv-2025.10-red) ![](https://img.shields.io/badge/ACL-2026-yellow)
- [LoLA: Low-Rank Linear Attention With Sparse Caching](https://arxiv.org/abs/2505.23666) ![](https://img.shields.io/badge/arXiv-2025.05-red)
- [STILL: Selecting Tokens for Intra-Layer Hybrid Attention to Linearize LLMs](https://arxiv.org/abs/2602.02180) ![](https://img.shields.io/badge/arXiv-2026.02-red)
- [Neural Attention Search Linear: Towards Adaptive Token-Level Hybrid Attention Models](https://arxiv.org/abs/2602.03681) ![](https://img.shields.io/badge/arXiv-2026.02-red)


<a id="contributing"></a>

## Contributing


This repository is intended to become a community-maintained living survey.

When adding or revising a paper entry, please include:

- **Title and link**
- **Venue and year**
- **Research line:** Softmax Attention, Sparse Attention, Linear Attention, State Space Models, or Hybrid Architecture
- **Subcategory:** select the most appropriate subcategory listed under the relevant survey section
- **Five-dimensional mapping:** Memory Representation, Memory Update, Access, Readout, and/or Integration
- **Rationale:** briefly explain the proposed placement and mapping using concrete evidence from the paper

If you identify missing work, incorrect metadata, broken links, or disagree with an existing classification, please open an issue or submit a pull request with the proposed change and supporting evidence. When modifying shared content, please keep `README.md` and `README-ZH.md` consistent.


<a id="citation"></a>

## Citation


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
