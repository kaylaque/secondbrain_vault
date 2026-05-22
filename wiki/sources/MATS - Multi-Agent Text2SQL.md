---
title: "MATS: Multi-Agent Text2SQL Framework using Small Language Models and Execution Feedback"
type: paper
tags: [text-to-sql, llm, slm, multi-agent, nlp, sql, reinforcement-learning, benchmarking]
date: 2026-05-22
source: "https://arxiv.org/html/2512.18622v1"
authors: ["Thanh Dat Hoang", "Thanh Trung Huynh", "Matthias Weidlich", "Thanh Tam Nguyen", "Tong Chen", "Hongzhi Yin", "Quoc Viet Hung Nguyen"]
year: 2025
venue: "arXiv preprint (arXiv:2512.18622)"
doi: ""
related: ["[[TriSQL - Text-to-SQL Framework with Dynamic Complexity Strategies]]", "[[Multi-Agent System Orchestration]]"]
status: seedling
confidence: low
---

# MATS: Multi-Agent Text2SQL Framework using Small Language Models

## In one sentence

MATS is a multi-agent framework for [[Text-to-SQL]] that achieves GPT-4o-level accuracy using a 9B-parameter pipeline of specialized small language models (SLMs), trained collaboratively via Reinforcement Learning with Execution Feedback (RLEF).

## Problem

Large LLM-based Text2SQL services (e.g., GPT-4o) raise **privacy** (schemas/query logs sent to third-party servers) and **cost** concerns that prevent enterprise adoption. Fine-tuned open-source models up to 15B params still underperform proprietary APIs. Small language models (SLMs, 100M–5B params) are efficient enough to run on a single GPU but lack the reasoning capacity for complex multi-table queries.

## Method

MATS decomposes the Text2SQL task across five specialized agents:

1. **Schema Investigator** — filters irrelevant schema elements; retrieves relevant column values from the database to disambiguate overlapping names
2. **Query Planner** — generates multiple SQL query candidates step-by-step using chain-of-thought
3. **Validator** — executes each candidate against the database; returns execution feedback (success/failure/result)
4. **Fix Agent** — refines SQL queries that failed validation, guided by error messages
5. **Selection Agent** — picks the best final SQL from the validated candidates

**RLEF (Reinforcement Learning with Execution Feedback):**
- Unlike [[RLHF]], RLEF uses database execution results (not human labels) as the reward signal
- Multiple responses are sampled; successful executions are positive examples; failed ones are negative
- Agents are trained jointly using preference optimization (DPO-style), aligning agents to collaborate effectively
- Training data extends Spider and BIRD datasets via manual labeling, few-shot prompting, and fine-tuning

## Key findings

- MATS at 9B total params matches GPT-4o + CoT and CHESS on BIRD dev (execution accuracy)
- Runs on a single GPU — practical for resource-constrained, privacy-sensitive deployments
- RLEF significantly improves accuracy over standard SFT alone, especially on complex queries
- Divide-and-conquer agent specialization compensates for individual SLM reasoning limitations

## Why it matters

Demonstrates that multi-agent collaboration + execution-feedback RL can close the accuracy gap between small open-source models and large proprietary LLMs for structured query generation. Directly relevant to any org that needs on-premise Text2SQL without data leaving the firewall.

## Limitations & caveats

- Evaluated primarily on Spider and BIRD (academic benchmarks); real-world schema diversity not fully tested
- 9B total params across agents — inference cost is additive; latency may be prohibitive for interactive use
- RLEF training requires a live database environment at train time — adds infrastructure complexity
- SLMs still lag on the hardest query types (deeply nested, multi-hop reasoning)

## My take

The privacy/cost motivation is compelling and underexplored in the Text2SQL literature. The RLEF idea is elegant — using the database itself as the judge eliminates the need for expensive human annotation and gives a clean, objective signal. The 5-agent decomposition mirrors how a human analyst might approach a complex query: schema → plan → validate → fix → select. Compare to [[TriSQL - Text-to-SQL Framework with Dynamic Complexity Strategies]]: TriSQL routes by complexity within a monolithic pipeline using large LLMs; MATS distributes roles across small agents. These are complementary strategies — MATS optimizes for resource constraints, TriSQL for accuracy on complex queries.

## Concepts introduced

- [[RLEF (Reinforcement Learning with Execution Feedback)]] — RL training using database execution results as reward signal instead of human labels
- [[MATS Framework]] — the five-agent Text2SQL architecture described in this paper

## Connects to

- [[TriSQL - Text-to-SQL Framework with Dynamic Complexity Strategies]] — alternative large-LLM Text2SQL approach; contrast: TriSQL = large LLMs + complexity routing; MATS = small LLMs + agent specialization
- [[Multi-Agent System Orchestration]] — MATS is a domain-specific MAS; the agent roles map to the worker/validator taxonomy in MAS literature
- [[Text-to-SQL]] — parent topic
- [[Reinforcement Learning from Execution Feedback]] — training paradigm
- [[Spider Benchmark]] — evaluation dataset
- [[BIRD Benchmark]] — evaluation dataset (harder; real-world databases with external knowledge)

## Citation

Hoang, T. D., Huynh, T. T., Weidlich, M., Nguyen, T. T., Chen, T., Yin, H., & Nguyen, Q. V. H. (2025). A multi-agent text2SQL framework using small language models and execution feedback. arXiv:2512.18622.
