---
title: "TriSQL: A Robust Natural Language Text-to-SQL Generation Framework with Dynamic Strategies Based on LLMs"
type: paper
tags: [text-to-sql, llm, nlp, sql, natural-language-processing, benchmarking, database]
date: 2026-05-20
source: "https://www.nature.com/articles/s41598-026-39128-9#Fig2"
authors: ["Xiaodong Su", "Yang Gu", "Peng Wang", "Wei Gu", "Lincheng Qi", "Jingwei He"]
year: 2026
venue: "Scientific Reports (Nature)"
doi: "10.1038/s41598-026-39128-9"
related: []
status: seedling
---

## In one sentence

TriSQL is a three-stage LLM-based framework for text-to-SQL generation that adapts its strategy to question complexity, achieving state-of-the-art execution accuracy on the Spider benchmark by combining schema selection, structure-aware generation, and complexity-aware refinement.

## Problem

Existing [[Text-to-SQL]] (Text2SQL) systems degrade sharply as question complexity increases. The core failure mode: most LLM-based methods apply a uniform generation strategy regardless of whether the question is simple or requires nested subqueries with multi-table joins. They also optimize for string-level exact match (EM) with gold SQL, which inflates EM scores while execution accuracy (EX) — the measure that actually matters for deployment — remains low. Models produce structurally plausible SQL that fails to run correctly on real databases.

## Method

TriSQL is a three-stage pipeline:

### Stage 1 — Question-Guided Schema Selector
- Computes cross-attention scores between the natural language question and each table/column in the database schema
- Table-level and column-level relevance are combined (with a tunable λ parameter) to produce a filtered schema subset S_ctx
- Keeps only the tables above a threshold τ, eliminating irrelevant schema noise before generation begins
- Trained with focal loss to handle class imbalance (relevant schema elements are sparse)

### Stage 2 — Structure-Aware SQL Generator
- Two-phase [[hierarchical decoding]]: first generate the SQL skeleton (SELECT, WHERE, GROUP BY order + typed placeholders), then fill placeholders with schema elements from S_ctx
- Placeholder positions are supervised with human-annotated masks during training; at inference, the model infers them from the question
- Separates structural validity from content grounding — simpler questions produce fewer placeholders, complex ones more

### Stage 3 — Complexity-Aware SQL Refiner
- A BERT-based classifier assigns a complexity level z ∈ {low, medium, high} to each (question, initial SQL, schema) triple
- **Low**: minor syntax corrections
- **Medium**: structural adjustments (clause reorganization, explicit JOIN rewrites, GROUP BY consistency)
- **High**: full LLM-driven semantic decomposition and rewriting
- Adaptive fallback: validates both the initial and refined SQL by executing them; selects the better one via a quality function Q(y) combining semantic alignment, structural plausibility, and execution efficiency
- If neither executes, escalates to z+1; if z=high fails, returns a UserError instead of invalid SQL
- Refiner is trained with supervised fine-tuning on complexity-labeled pairs, then reinforcement learning using execution success as reward signal

### Training
Multi-stage: each module trained separately with tailored objectives, then jointly fine-tuned with complexity-aware data augmentation.

## Key findings

- TriSQL achieves **82.2% EX and 76.4% EM** on Spider test set — highest EX among reported methods
- Robustness score R = 0.745 vs. RESDSQL (0.601) and DIN-SQL (0.507); robustness is defined as average EX minus average complexity-level degradation
- At extra-high complexity: TriSQL 76% EX vs. RESDSQL 58%, DIN-SQL 48%
- Spider-DK: 62.42 EX (+18.4% over best baseline)
- Spider-Syn: 77.16 EX (best)
- Spider-Realistic: 72.63 EX (best)
- Also evaluated on **PowerSQL**, a proprietary power-grid domain benchmark (3,427 queries including INSERT/UPDATE/DELETE/CREATE, 58-table schema), validating beyond SELECT-only settings

## Why it matters

The key insight — that uniform generation strategies are the bottleneck, not model capacity — motivates the architectural choice to route queries through different refinement intensities. This is a cleaner framing than just scaling up LLM prompts. The paper also makes a good case for prioritizing EX over EM as the primary evaluation metric, since high EM with low EX is practically useless for deployment. The PowerSQL benchmark contribution is useful for evaluating non-SELECT SQL types in real industrial settings.

## Limitations & caveats

- Experiments focus on Spider and variants; generalization to more diverse real-world schemas (beyond 200 databases) is not deeply analyzed
- The three-stage pipeline adds architectural complexity and inference latency; efficiency analysis is done but the overhead of complexity classification and adaptive refinement may be prohibitive in low-latency settings
- PowerSQL comes from a single organization (State Grid Jiangsu), limiting claims about general real-world robustness
- Scope is limited to SQL (relational); the paper briefly notes NoSQL and graph query languages as future directions but does not address them
- The BERT-based complexity classifier is a potential bottleneck — errors in complexity assignment propagate through the refinement stage

## My take

The three-stage design is well-motivated and the robustness framing (not just aggregate accuracy but accuracy degradation curve across complexity bins) is a genuinely useful contribution to how we evaluate Text2SQL systems. The adaptive fallback with complexity escalation is a practical engineering insight. Worth revisiting when building any system that routes NL queries to SQL over large, messy schemas. The RL training on execution feedback is now a fairly standard component in this space, but the combination with complexity-aware routing is novel here.

## Concepts introduced

- [[TriSQL]] — the three-stage framework proposed in this paper
- [[Complexity-Aware SQL Refinement]] — adapting refinement strategy based on classified query difficulty
- [[Question-Guided Schema Selection]] — cross-attention-based filtering of irrelevant schema elements before generation
- [[Hierarchical SQL Decoding]] — skeleton-first, then placeholder-fill generation
- [[Robustness Score (Text2SQL)]] — average EX minus average per-level degradation, defined in this paper
- [[PowerSQL]] — proprietary power-grid SQL benchmark with multi-operation types (SELECT/INSERT/UPDATE/DELETE/CREATE)
- [[Execution Accuracy vs Exact Match]] — key distinction: EM measures string similarity to gold SQL; EX measures whether results match

## Connects to

- [[Spider Benchmark]] — primary evaluation dataset
- [[DIN-SQL]] — baseline; uses difficulty-aware prompting but uniform strategy
- [[RESDSQL]] — baseline; decouples schema linking from generation but uses static relevance
- [[RAT-SQL]] — earlier relation-aware schema encoder approach
- [[NatSQL]] — intermediate SQL representation used by several baselines
- [[Schema Linking]] — the general problem of grounding NL to database schema elements
- [[Text-to-SQL]] — parent topic
- [[Reinforcement Learning from Execution Feedback]] — training paradigm used in Stage 3
- [[Focal Loss]] — used in schema selector training to handle class imbalance

## Citation

Su, X., Gu, Y., Wang, P., Gu, W., Qi, L., & He, J. (2026). A robust natural language text-to-SQL generation framework with dynamic strategies based on LLMs. *Scientific Reports*. https://doi.org/10.1038/s41598-026-39128-9
