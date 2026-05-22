---
title: "LLM Wiki Pattern"
type: concept
tags: [pkm, llm, knowledge-management, second-brain, rag, obsidian, workflow]
date: 2026-05-22
source: "https://agentpedia.codes/blog/karpathy-llm-wiki-idea-file"
related: ["[[Andrej Karpathy]]", "[[Vivian Balakrishnan - Second Brain Talk]]"]
status: seedling
confidence: medium
---

# LLM Wiki Pattern

## What it is

A knowledge management architecture in which an LLM agent incrementally builds and maintains a structured, interlinked wiki of markdown files from raw source documents — instead of re-deriving answers from raw sources on every query (as RAG does). Introduced as an "idea file" by [[Andrej Karpathy]] in April 2026.

## Why it matters

Traditional RAG re-discovers knowledge from scratch on every question. The LLM Wiki compiles knowledge **once at ingest time** and keeps it current — cross-references are pre-built, contradictions are flagged, and synthesis compounds with every new source. Humans abandon wikis because maintenance burden grows faster than value; LLMs handle the bookkeeping without getting bored.

## How it works

**Three-layer architecture:**

| Layer | Contents | Owner |
|-------|----------|-------|
| `raw/` | Immutable source documents (articles, papers, PDFs) | Human (never modified) |
| `wiki/` | LLM-generated markdown pages (concepts, entities, comparisons, source summaries) | LLM |
| `CLAUDE.md` / `AGENTS.md` | Schema: page conventions, frontmatter, workflows | Co-evolved by human + LLM |

**Three operations:**

1. **Ingest** — drop a source into `raw/`, LLM reads it, creates/updates wiki pages, flags contradictions, updates index and log
2. **Query** — LLM reads `index.md` to find relevant pages, reads them, synthesizes an answer with `[[wikilink]]` citations; good answers get filed back as new wiki pages (compounding loop)
3. **Lint** — periodic health check: contradictions, orphan pages, missing concept pages, stale claims, suggested investigations

**Navigation without RAG:** At moderate scale (~100 pages), a well-maintained `index.md` catalog is sufficient — LLM reads the index, identifies relevant pages, reads those directly. `qmd` (BM25 + vector + LLM re-ranking, all local) is added when the index outgrows a single context window.

## Key distinctions

- Not [[RAG]] — RAG re-derives at query time; LLM Wiki compiles once at ingest
- Not a static wiki — the LLM maintains it; humans only source and question
- Not a database — plain markdown files, git-versioned, human-readable without any tooling

## Examples

- Karpathy's own ML research wiki (~100 articles, ~400,000 words on a single topic)
- This vault (secondbrain_vault) — implements the pattern with `CLAUDE.md` as schema, `wiki/` flat structure, three operations defined

## Open questions

- At what wiki size does `index.md` navigation break down and `qmd` become necessary?
- How should contradictions between pages of different `confidence` levels be resolved?
- How to handle multi-topic vaults where different domains rarely cross-link?

## Sources

- [[Andrej Karpathy]] — originator; GitHub gist at gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- Agentpedia.codes breakdown article (2026-04-04)

## Connects to

- [[Andrej Karpathy]] — originator of the pattern
- [[Vivian Balakrishnan - Second Brain Talk]] — independent convergence on same philosophy (personal, curated, AI-maintained knowledge)
- [[RAG]] — contrasted approach
- [[Memex]] — Vannevar Bush's 1945 vision that inspired the pattern
