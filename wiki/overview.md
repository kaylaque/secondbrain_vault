---
title: "Vault Overview"
type: concept
tags: [meta, pkm, bioinformatics, ai-agents, text-to-sql, bgc]
date: 2026-05-22
status: growing
---

# Vault Overview

This vault clusters around two research areas that currently have no direct connection but share a common thread: **using AI to manage and reason over complex, structured knowledge**.

---

## Cluster 1 — Bioinformatics & BGC Tooling

Focused on biosynthetic gene cluster (BGC) discovery, annotation, and curation.

Key pages: [[antiSMASH 8.0]], [[antiSMASH Database v4]], [[MIBiG 4.0]], [[BGCFlow]]

- **antiSMASH** is the core detection engine — identifies BGC types from genome sequences
- **antiSMASH Database** is the indexed output at scale (231K+ BGCs)
- **MIBiG** is the curated reference layer — experimentally validated BGCs with biosynthetic pathway annotations
- **BGCFlow** is the workflow layer — Snakemake pipeline integrating all tools at pangenome scale
- **BioAgent Bench** evaluates AI agents on bioinformatics tasks, including the kind of pipeline work BGCFlow automates

This cluster is mature tooling: each piece has a well-defined role, and the integration pattern (detect → annotate → curate → compare) is established.

---

## Cluster 2 — AI Agents, LLMs & Knowledge Management

Focused on agentic AI architectures, Text-to-SQL systems, and personal knowledge management.

Key pages: [[Multi-Agent System Orchestration]], [[MATS - Multi-Agent Text2SQL]], [[TriSQL - Text-to-SQL Framework with Dynamic Complexity Strategies]], [[BioAgent Bench]], [[LLM Wiki Pattern]], [[Vivian Balakrishnan - Second Brain Talk]]

- **MAS Orchestration** provides the architectural blueprint (orchestration layer, MCP + A2A protocols, worker/service/support agent taxonomy)
- **MATS** and **TriSQL** are complementary Text-to-SQL approaches: MATS optimizes for privacy-sensitive on-premise SLM deployment; TriSQL optimizes for accuracy on complex queries using large LLMs
- **MLflow** is the observability layer for LLM experiments and agent evaluation
- **LLM Wiki Pattern** (Karpathy) and **Vivian Balakrishnan's Second Brain Talk** are the meta-layer: how to use AI to maintain knowledge, which is what this vault itself instantiates

---

## The connecting thread

Both clusters involve **AI agents operating over structured, domain-specific knowledge** — whether that's genomic databases or SQL schemas. The knowledge management cluster is also reflexive: this vault is itself an implementation of the LLM Wiki Pattern.

**Priority concept pages to create** (from lint 2026-05-22):
- [[Biosynthetic Gene Cluster]] — needed by 4 bioinformatics pages
- [[Text-to-SQL]] — needed by TriSQL + MATS
- [[Multi-agent System]] — needed by MAS Orchestration + BioAgent Bench + MATS
