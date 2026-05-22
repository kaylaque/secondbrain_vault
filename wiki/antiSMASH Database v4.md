---
title: "antiSMASH Database v4"
type: paper
tags: [bgc, bioinformatics, genomics, antismash, mibig, natural-products, secondary-metabolism]
date: 2026-05-20
source: "_attachments/The antiSMASH database version 4- additional genomes and BGCs, new sequence-based searches and more.pdf"
authors: ["Kai Blin", "Simon Shaw", "Marnix H Medema", "Tilmann Weber"]
year: 2024
venue: "Nucleic Acids Research"
doi: "10.1093/nar/gkad984"
related: ["[[antiSMASH]]", "[[BGC]]", "[[MIBiG]]", "[[RiPP]]", "[[Genome Mining]]"]
status: seedling
---

# antiSMASH Database v4

## In one sentence

Version 4 of the antiSMASH database expands to 231,534 high-quality [[Biosynthetic Gene Cluster]]s from 36,554 archaeal, bacterial, and fungal genomes, and introduces sequence-based searches for [[RiPP]] precursors and arbitrary protein sequences.

## Problem

Cross-genome comparison of [[Biosynthetic Gene Cluster]]s predicted by [[antiSMASH]] requires a centralized, curated database that handles redundancy in NCBI RefSeq (caused by thousands of near-identical pathogen genomes and fragmented draft assemblies) and enables complex multi-faceted queries — something that single-genome analysis alone cannot provide.

## Method

- Archaeal, bacterial, and fungal genomes downloaded from [[NCBI RefSeq]] (April 2023) at complete, chromosome, and scaffold assembly levels using `ncbi-genome-download`.
- Assemblies with >100 contigs (archaea/bacteria) or >150 contigs (fungi) discarded to avoid fragmentation artifacts in [[BGC]] detection.
- Sequence-similarity redundancy filtering applied using [[Mash]] across all taxa (including fungi, a new addition over v3).
- [[antiSMASH]] 7.1 run with full annotation options (`--cb-knownclusters`, `--cb-subclusters`, `--cc-mibig`, `--clusterhmmer`, `--tigrfam`, `--pfam2go`, `--rre`, `--asf`, `--tfbs`).
- Two-pass annotation: first pass extracts all predicted [[RiPP]] precursors to build updated [[CompaRiPPson]] and [[ClusterBlast]] datasets; second pass re-annotates with `--cb-general --reuse`.
- New sequence-based searches: blastp for [[RiPP]] precursor queries; [[DIAMOND]] for arbitrary protein sequence queries against all BGC proteins.
- Background job processing introduced to replace the previous synchronous download model, making large-slice downloads reliable.

## Key findings
- Database grew from 25,802 assemblies (v3) to 36,554 (v4) — a ~42% increase.
- BGCs not cut by contig edges increased from 147,517 to 231,534 — a ~57% increase.
- antiSMASH 7.1 supports 88 distinct pathway types, including newly added categories: isocyanides, NRP-related isocyanides, highly-reducing PKS type IIs, darobactins, triceptides, archaeal [[RiPP]]s, and hydrogen cyanides.
- Simple text search removed; all queries now go through the structured query builder, which covers 39 search categories.
- Two new sequence-based search modes: [[RiPP]] precursor search (blastp) and protein sequence search ([[DIAMOND]]).
- Background download jobs replace synchronous pagination, allowing automated scripts to retrieve large dataset slices reliably.
- Complete dataset (antiSMASH JSON files and SQL dump) available for bulk download at `dl.secondarymetabolites.org/database/4.0/`.

## Why it matters

The antiSMASH database is the primary resource enabling genome-scale comparison of [[Biosynthetic Gene Cluster]]s across the microbial tree of life. Version 4's sequence-based search capabilities lower the barrier for researchers to connect their own protein or peptide sequences directly to catalogued BGC diversity, accelerating [[natural product]] discovery and BGC dereplication without requiring local antiSMASH installations.

## Limitations & caveats

- Coverage is limited to [[NCBI RefSeq]] complete/chromosome/scaffold assemblies; contig-level assemblies and assemblies exceeding fragmentation thresholds are excluded, potentially missing BGC diversity in poorly assembled organisms.
- Redundancy filtering reduces representation of highly sequenced species (e.g., *E. coli*, *Salmonella*, *S. aureus*), which may matter for population-level BGC studies.
- Annotations reflect antiSMASH 7.1 rule sets; BGC types supported by earlier or later versions are not back- or forward-annotated automatically.
- Fungal coverage (236 assemblies) remains far smaller than bacterial coverage (35,726 assemblies), limiting fungal [[secondary metabolism]] analysis.

## My take

This is primarily an infrastructure and scale paper rather than a methodological advance — the core value is in the updated, larger, and more queryable dataset. The sequence-based search addition (especially the protein sequence search via [[DIAMOND]]) is the most practically useful new feature for everyday genome mining work. The removal of the free-text search in favor of the structured query builder is a reasonable UX trade-off given 39 categories, but may frustrate users accustomed to quick keyword lookups. Worth revisiting when doing any comparative BGC analysis across taxa.

## Concepts introduced
- [[CompaRiPPson]] — cross-database comparison tool for [[RiPP]] precursor peptides, now updated with the full database precursor set.
- [[ClusterBlast]] — antiSMASH module that compares detected BGCs to the database; hits link directly to antiSMASH database entries.
- [[Mash]] — MinHash-based sequence similarity tool, now used for redundancy filtering across all taxa including fungi.

## Connects to
- [[antiSMASH]]
- [[Biosynthetic Gene Cluster]]
- [[MIBiG]]
- [[RiPP]]
- [[Genome Mining]]
- [[NCBI RefSeq]]
- [[DIAMOND]]
- [[Natural Products]]

## Quotes worth keeping
> "With a selection of 231,534 BGC regions from archaea, bacteria and fungi, the antiSMASH database version 4 is a comprehensive collection of secondary/specialised metabolite biosynthetic gene clusters with up-to-date, high quality antiSMASH-based annotations available to the natural product research community."

> "Compared to version 3's 25,802 assemblies, version 4 contains 36,554, roughly a 42% increase. At the same time, the number of high-quality BGCs increased... from 147,517 to 231,534, almost 57%."

## Citation

Blin, K., Shaw, S., Medema, M.H., & Weber, T. (2024). The antiSMASH database version 4: additional genomes and BGCs, new sequence-based searches and more. *Nucleic Acids Research*, 52, Database issue, D587–D589. https://doi.org/10.1093/nar/gkad984

---
*Processed: 2026-05-20*
