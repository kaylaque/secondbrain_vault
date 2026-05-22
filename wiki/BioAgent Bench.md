---
title: "BioAgent Bench"
type: paper
tags: [ai-agents, bioinformatics, benchmarking, llm, evaluation]
date: 2026-05-20
source: "https://arxiv.org/html/2601.21800v2"
authors: ["Dionizije Fa", "Marko Čuljak", "Bruno Pandža", "Mateo Čupić"]
year: 2026
venue: "ICML"
doi: ""
related: ["[[Bioinformatics Pipeline]]", "[[LLM Benchmark]]", "[[AI Agent]]"]
status: seedling
---

# BioAgent Bench

## In one sentence

A benchmark suite of 10 end-to-end [[bioinformatics]] tasks used to evaluate and compare frontier [[AI Agent|AI agents]] on realistic genomics workflows, revealing that pipeline completion rates overstate true reliability.

## Problem

There was no rigorous, standardized way to evaluate how well [[AI Agent|AI agents]] handle real-world [[bioinformatics]] pipelines — tasks like [[RNA-seq]] analysis, [[variant calling]], and [[metagenomics]] that require multi-step reasoning, tool use, and domain knowledge. Existing [[LLM Benchmark|benchmarks]] did not capture end-to-end execution or expose failure modes specific to life-sciences workflows.

## Method

The authors constructed a benchmark dataset of 10 end-to-end bioinformatics tasks spanning [[RNA-seq]], [[variant calling]], [[metagenomics]], [[comparative genomics]], and [[single-cell analysis]]. Models were evaluated across three agentic harnesses — [[Claude Code]], [[Codex CLI]], and [[OpenCode]] — using both frontier closed-source models and open-weight models. An [[LLM Grader]] (GPT-5.1) scored each run on pipeline progress and outcome validity. Robustness was tested via corrupted inputs, decoy files, and prompt bloat. Stability was measured with [[Jaccard Index]] and [[Pearson correlation]] across repeated trials.

## Key findings
- [[Claude Opus 4.5]] achieved 100% pipeline completion; [[GPT-5.2]], [[Gemini 3 Pro]], and [[Sonnet 4.5]] exceeded 90%
- Open-weight [[GLM-4.7]] reached 82.5% completion, demonstrating viability for privacy-sensitive settings
- Robustness probes: models detected corrupted inputs in 7/10 cases, failed on decoy files in 2/10 cases, and [[prompt bloat]] caused a −28% drop in completion rate
- [[Jaccard Index]] of 0.43 and [[Pearson correlation]] of 0.73 across trials indicate significant run-to-run variability
- Pipeline completion rate is a misleading metric — step-level reasoning can still fail even when the full pipeline nominally completes

## Why it matters

[[AI Agent|AI agents]] are increasingly being applied to genomics and clinical research workflows. [[BioAgent Bench]] provides the first systematic evaluation framework for this setting, exposing that headline completion metrics mask fragile intermediate reasoning. The inclusion of open-weight models matters because [[clinical genomics]] and sensitive research settings require on-premise, privacy-preserving deployment that closed APIs cannot offer.

## Limitations & caveats

- Only 10 tasks — coverage of the full [[bioinformatics]] landscape is limited
- Stability metrics (Jaccard 0.43) suggest results may not generalize reliably across runs or environments
- The [[LLM Grader]] (GPT-5.1) introduces its own model bias into scoring
- Benchmark reflects a snapshot of models available at time of publication; frontier models change rapidly

## My take

The core insight — that completion rate overestimates reliability — applies broadly to [[agentic evaluation]] beyond bioinformatics. The robustness probes (decoy files, prompt bloat) are a useful methodological contribution worth borrowing for other domains. The variability numbers (Jaccard 0.43) are a red flag for anyone considering deploying these agents in production clinical pipelines without human oversight.

## Concepts introduced
- [[BioAgent Bench]] — standardized benchmark suite of 10 end-to-end bioinformatics agentic tasks with robustness and stability evaluation
- [[LLM Grader]] — use of a separate LLM (GPT-5.1) to score pipeline progress and outcome validity in lieu of deterministic metrics
- [[Prompt Bloat]] — robustness probe adding irrelevant text to prompts, causing −28% completion rate drop in tested agents

## Connects to
- [[AI Agent]]
- [[LLM Benchmark]]
- [[Bioinformatics Pipeline]]
- [[RNA-seq]]
- [[Variant Calling]]
- [[Metagenomics]]
- [[Single-Cell Analysis]]
- [[Claude Code]]
- [[Agentic Evaluation]]

## Quotes worth keeping
> Pipeline completion overestimates reliability; step-level reasoning can still fail.

## Citation
Fa et al. (2026). BioAgent Bench: An AI Agent Evaluation Suite for Bioinformatics. ICML. https://arxiv.org/html/2601.21800v2

---
*Processed: 2026-05-20*
