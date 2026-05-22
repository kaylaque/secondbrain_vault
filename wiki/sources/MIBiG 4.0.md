---
title: "MIBiG 4.0"
type: paper
tags: [bgc, bioinformatics, mibig, natural-products, secondary-metabolism, curation]
date: 2026-05-20
source: "_attachments/MIBiG 4.0- advancing biosynthetic gene cluster curation through global collaboration.pdf"
authors: ["Zdouc, Mitja M.", "Blin, Kai", "Louwen, Nico L.L.", "Navarro, Jorge", "Loureiro, Catarina", "Weber, Tilmann", "Medema, Marnix H.", "et al. (267 contributors total)"]
year: 2024
venue: "Nucleic Acids Research"
doi: "10.1093/nar/gkae1115"
related: ["[[antiSMASH]]", "[[BGC]]", "[[Natural Products]]", "[[Secondary Metabolism]]", "[[Genome Mining]]"]
status: seedling
---

# MIBiG 4.0

## In one sentence

MIBiG 4.0 is a major community-driven update to the [[Minimum Information about a Biosynthetic Gene Cluster]] data standard and repository, adding 557 new entries and modifying 590 existing ones to reach 3,059 curated [[Biosynthetic Gene Cluster]] entries, with new data quality controls and a peer-review model.

## Problem

Specialized (secondary) metabolites — natural products with potent biological activities relevant to agriculture, engineering, and medicine — are encoded by [[Biosynthetic Gene Cluster]]s (BGCs) in microbial and fungal genomes. As genome mining tools like [[antiSMASH]] proliferate, the field needs a standardized, machine-readable, high-quality reference database of experimentally characterized BGCs to benchmark predictions, train models, and share knowledge consistently across the research community. Prior MIBiG versions were curated by a small core team, limiting throughput and coverage.

## Method

A massive global community annotation effort engaged 267 contributors who collectively performed 8,304 edits to the MIBiG repository. The update introduced:
- A custom submission portal prototype for structured, web-based BGC data entry
- Automated data validation pipelines to enforce schema compliance and flag inconsistencies
- A novel peer-reviewing model where entries are checked by independent curators before acceptance
- Expansion of the underlying [[MIBiG data standard]] (JSON schema) to capture richer metadata including biosynthetic logic, compound activities, and taxonomic information
- Steps toward a rolling release model to allow continuous integration of new entries rather than batch releases

## Key findings
- The repository grew from 2,502 (v3.1) to 3,059 curated BGC entries — a 22% increase
- 557 entirely new BGC entries were created; 590 existing entries were substantively revised
- 267 contributors from across the globe participated, making this the most community-involved MIBiG release to date
- Automated validation paired with peer review substantially improved data consistency and quality compared to previous versions
- The new submission portal lowers the barrier for expert contributors to deposit curated BGC data
- MIBiG 4.0 remains the primary reference dataset for training and benchmarking [[Genome Mining]] tools such as [[antiSMASH]] and [[BiG-SCAPE]]

## Why it matters

MIBiG is the de facto gold-standard reference for experimentally validated BGCs. A larger, higher-quality MIBiG directly improves the accuracy and coverage of computational [[Genome Mining]] tools, enabling better discovery of novel natural products with pharmaceutical, agricultural, and industrial value. The shift toward community curation and rolling releases makes the database more scalable and responsive to the pace of natural product research.

## Limitations & caveats

- Despite 3,059 entries, MIBiG covers only a small fraction of the estimated diversity of BGCs across microbial life; coverage is heavily biased toward bacteria (especially actinomycetes) and well-studied compound classes
- Community curation quality depends on contributor expertise; the peer-review model mitigates but does not eliminate errors
- The rolling release model is described as a goal rather than a fully implemented system in this version
- Taxonomic and functional metadata completeness varies across entries, which can affect downstream analyses

## My take

MIBiG 4.0 is an important infrastructure paper for the natural products field. The shift from small-team curation to community-scale peer-reviewed annotation is the right approach and mirrors successful models in structural biology (PDB) and genomics. The key technical contribution is the submission portal and automated validation — these are what make community curation tractable at scale. For anyone doing BGC discovery or comparative genomics of secondary metabolite pathways, MIBiG 4.0 is the reference dataset to use. The DOI-linked abstract and data standard versioning make it suitable as a stable citation target.

## Concepts introduced
- [[Minimum Information about a Biosynthetic Gene Cluster]] (MIBiG) — a data standard and repository defining the minimum metadata required to describe an experimentally characterized BGC
- [[Rolling Release Model]] — a database update strategy allowing continuous, validated additions rather than periodic batch releases
- [[BGC Curation Portal]] — a web-based submission interface with automated schema validation enabling community members to contribute structured BGC annotations
- [[Peer-Review Curation Model]] — an independent review step applied to community-submitted BGC entries before they are accepted into the repository

## Connects to
- [[MIBiG]]
- [[Biosynthetic Gene Cluster]]
- [[antiSMASH]]
- [[BiG-SCAPE]]
- [[Genome Mining]]
- [[Natural Products]]
- [[Secondary Metabolism]]

## Quotes worth keeping
> "In a massive community annotation effort, 267 contributors performed 8304 edits, creating 557 new entries and modifying 590 existing entries, resulting in a new total of 3059 curated entries in MIBiG."

> "Particular attention was paid to ensuring high data quality, with automated data validation using a newly developed custom submission portal prototype, paired with a novel peer-reviewing model."

## Citation

Zdouc MM, Blin K, Louwen NLL, Navarro J, Loureiro C, et al. (2024). MIBiG 4.0: advancing biosynthetic gene cluster curation through global collaboration. *Nucleic Acids Research*, Database issue. https://doi.org/10.1093/nar/gkae1115

---
*Processed: 2026-05-20*
