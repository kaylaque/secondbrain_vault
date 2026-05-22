---
title: "BGCFlow"
type: paper
tags: [bgc, bioinformatics, pangenome, genomics, antismash, mibig, natural-products, secondary-metabolism, workflow, snakemake]
date: 2026-05-20
source: "_attachments/BGCFlow- systematic pangenome workflow for the analysis of biosynthetic gene clusters across large genomic datasets.pdf"
authors: ["Matin Nuhamunada", "Omkar S. Mohite", "Patrick V. Phaneuf", "Bernhard O. Palsson", "Tilmann Weber"]
year: 2024
venue: "Nucleic Acids Research"
doi: "10.1093/nar/gkae314"
related: ["[[antiSMASH]]", "[[BiG-SCAPE]]", "[[MIBiG]]", "[[BiG-SLICE]]", "[[ARTS2]]", "[[Snakemake]]"]
status: seedling
---

# BGCFlow

## In one sentence

BGCFlow is a Snakemake-based workflow that integrates genome mining, functional annotation, phylogenetic analysis, and comparative genomics tools into a single reproducible platform for large-scale [[Biosynthetic Gene Cluster]] discovery across bacterial [[Pangenome|pangenomes]].

## Problem

The rapid growth of publicly available bacterial genomes has outpaced researchers' ability to manually extract [[Secondary Metabolism|biosynthetic]] knowledge from them. Existing tools like [[antiSMASH]], [[BiG-SCAPE]], and [[MIBiG]] each address one piece of the puzzle but lack integration, requiring significant domain expertise to chain them together in a reproducible, scalable manner. No end-to-end platform existed for systematic pangenome-wide [[Biosynthetic Gene Cluster]] analysis.

## Method

BGCFlow wraps ~70 [[Snakemake]] rules into five sequential (but re-entrant) analysis stages:

1. **Data selection** — genome quality filtering with CheckM, Seqfu, MASH, FastANI; taxonomic placement with GTDB-Tk; input via [[Portable Encapsulated Projects (PEP)]] YAML configs pointing to NCBI/PATRIC accessions or local FASTA files.
2. **Functional annotation** — gene calling with Prokka/Prodigal, ortholog prediction with EggNOG-mapper (covering COG, KEGG, GO), pangenome reconstruction with Roary, and a deep-learning transcription factor predictor (deepTFactor).
3. **Phylogenetic analysis** — multi-locus sequence tree via autoMLST, core-genome tree via Roary, MASH/FastANI distance matrices.
4. **Genome mining** — [[antiSMASH]] (v6/v7) for BGC detection; GECCO as a machine-learning alternative; [[ARTS2]] for resistance-gene-proximity prioritization.
5. **Comparative analysis** — [[BiG-SCAPE]] for [[Gene Cluster Family (GCF)]] networks; [[BiG-SLICE]] queries against BiG-FAM (~1.2 M public BGCs); clinker for BGC alignment visualization.

Results are stored as Apache Parquet files, transformed via dbt into a [[DuckDB]] OLAP database, and served through Metabase (interactive SQL) and MkDocs (static HTML reports with embedded Jupyter Notebooks).

## Key findings

- BGCFlow was applied to 42 *Saccharopolyspora* genomes, detecting 722 BGCs across eight phylogroups.
- Integration of KnownClusterBlast, BiG-SLICE, and BiG-SCAPE expanded connected BGC similarity network components from 330 to 206 while reducing singletons from 209 to 51, demonstrating more accurate dereplication.
- Targeted comparative analysis of [[RiPP|RiPPs]] revealed novel erythreapeptin-like lanthipeptide variants with distinct precursor peptide sequences across phylogroups.
- Putative mycofactocin-like [[Ranthipeptide]] BGCs were identified in *Saccharopolyspora* — a cluster type previously only described in *Mycobacterium tuberculosis* — through ARTS2 integration (TIGR03997 proximity hits).
- The five-command CLI (deploy → init → run → build → serve) reduces the barrier to end-to-end genome mining studies without sacrificing reproducibility.

## Why it matters

As bacterial genome databases grow exponentially, the bottleneck in natural products discovery shifts from sequencing to interpretation. BGCFlow provides a FAIR-compliant (Findable, Accessible, Interoperable, Reproducible) infrastructure that any lab can deploy to systematically survey the biosynthetic potential of any bacterial genus, accelerating prioritization of novel [[Natural Products|natural product]] leads and reducing duplicated bioinformatics effort across the field.

## Limitations & caveats

- Accuracy of [[Biosynthetic Gene Cluster]] detection is bounded by the underlying tools (antiSMASH, GECCO); fragmented, low-quality assemblies inflate edge-truncated BGC counts and reduce completeness.
- The five-stage pipeline is designed for bacterial genomes; fungal or other eukaryotic datasets are not supported out of the box.
- The interactive database and Metabase server add infrastructure overhead that may be impractical for very small projects or resource-constrained environments.
- Default parameters (e.g., BiG-SCAPE 0.30 similarity cutoff) may need tuning for taxa with unusual BGC diversity or highly conserved BGC classes.

## My take

BGCFlow fills a genuine gap: it is one of the first tools to treat pangenome-scale genome mining as a first-class workflow engineering problem rather than an ad hoc scripting exercise. The decision to use Snakemake (rather than a custom engine) means the community can extend it with new rules as tools evolve. The DuckDB + Metabase layer is a smart move for making results explorable by non-programmers. The *Saccharopolyspora* case study is convincing but modest in scale (42 genomes); future benchmarks on thousands of genomes would strengthen the scalability claims.

## Concepts introduced

- [[Portable Encapsulated Projects (PEP)]] — a YAML-based configuration format used by BGCFlow to define genome datasets and rule selections per project, enabling multi-project management in a single workflow instance.
- [[Gene Cluster Family (GCF)]] — groupings of BGCs with similar sequence composition used for dereplication and comparative analysis, computed here by BiG-SCAPE and queried against BiG-FAM.
- [[Re-entrant Workflow]] — the design property of BGCFlow allowing analysts to restart or subset any analysis stage without rerunning the entire pipeline, critical for iterative exploratory genome mining.

## Connects to

- [[Biosynthetic Gene Cluster]]
- [[Pangenome]]
- [[antiSMASH]]
- [[BiG-SCAPE]]
- [[MIBiG]]
- [[Natural Products]]
- [[Secondary Metabolism]]
- [[Snakemake]]
- [[RiPP]]

## Quotes worth keeping

> "The scalable, interoperable, adaptable, re-entrant, and reproducible nature of the BGCFlow will provide an effective novel way to extract the biosynthetic knowledge from the ever-growing genomic datasets of biotechnologically relevant bacterial species."

> "A major limitation of many bioinformatics workflows is the lack of interoperability between different tools and analyses. BGCFlow provides an effective solution for investigating the BGCs spread across the dataset by integrating various bioinformatics tools for genomic analyses."

## Citation

Nuhamunada M, Mohite OS, Phaneuf PV, Palsson BO, Weber T. BGCFlow: systematic pangenome workflow for the analysis of biosynthetic gene clusters across large genomic datasets. *Nucleic Acids Research*. 2024;52(10):5478–5495. https://doi.org/10.1093/nar/gkae314

---
*Processed: 2026-05-20*
