---
title: "antiSMASH 8.0"
type: paper
tags: [bgc, bioinformatics, antismash, secondary-metabolism, natural-products, genomics, mibig]
date: 2026-05-20
source: "_attachments/antiSMASH 8.0- extended gene cluster detection capabilities and analyses of chemistry, enzymology, and regulation.pdf"
authors: ["Kai Blin", "Simon Shaw", "Lisa Vader", "Judit Szenei", "Zachary L Reitz", "Hannah E Augustijn", "José D D Cediel-Becerra", "Valérie de Crécy-Lagard", "Robert A Koetsier", "Sam E Williams", "Pablo Cruz-Morales", "Sopida Wongwas", "Alejandro E Segurado Luchsinger", "Friederike Biermann", "Aleksandra Korenskaia", "Mitja M Zdouc", "David Meijer", "Barbara R Terlouw", "Justin J J van der Hooft", "Nadine Ziemert", "Eric J N Helfrich", "Joleen Masschelein", "Christophe Corre", "Marc G Chevrette", "Gilles P van Wezel", "Marnix H Medema", "Tilmann Weber"]
year: 2025
venue: "Nucleic Acids Research"
doi: "10.1093/nar/gkaf334"
related: ["[[antiSMASH]]", "[[Biosynthetic Gene Cluster]]", "[[MIBiG]]", "[[Natural Products]]", "[[NRPS]]", "[[PKS]]"]
status: seedling
---

# antiSMASH 8.0

## In one sentence

antiSMASH 8.0 expands automated [[Biosynthetic Gene Cluster]] detection from 81 to 101 cluster types and introduces dedicated analysis modules for terpenoids, tailoring enzymes, and transcription factor binding sites, keeping the platform current with the rapidly evolving natural products field.

## Problem

Microbial genomes encode a vast and largely unexplored reservoir of [[Secondary Metabolite|secondary metabolites]] with pharmaceutical and agrochemical relevance. Manually identifying and characterizing [[Biosynthetic Gene Cluster|BGCs]] at genome scale is intractable, and prior antiSMASH versions lacked detection rules for newly characterised cluster chemistries and offered limited support for terpenoid classification, tailoring enzyme annotation, and regulatory analysis.

## Method

The tool performs [[Hidden Markov Model|profile HMM]]-based detection of BGC signature genes across input nucleotide sequences (bacterial and fungal genomes or metagenomes). Version 8 adds:
- **20 new cluster-type detection rules** covering archaeal [[RiPP|RiPPs]], atropopeptides, nitropropanoic acid, azoxy compounds, polyynes, deazapurines, and more.
- A rebuilt **terpene analysis module** using curated pHMMs to predict terpenoid class, chain length, subclass, and product name.
- A **tailoring enzyme module** organising enzymes by [[Enzyme Commission]] category and cross-linking to the [[MITE Database]].
- Expanded [[NRPS]]/[[PKS]] domain detection (β-hydroxylases, interface domains, CAL domains, active-site checking for condensation and epimerization domains).
- Updated [[KnownClusterBlast]] and [[ClusterCompare]] datasets against [[MIBiG]] 4.0.
- Extended transcription factor binding site predictions using the [[CollecTF]] database.
- Proper handling of BGCs that span the origin of replication in circular genomes.

## Key findings
- Detection capacity expanded from 81 to 101 [[Biosynthetic Gene Cluster|BGC]] types, the largest single-version jump in cluster-type coverage.
- New terpene module predicts terpenoid subclass and likely product from genomic sequence alone, restoring and greatly improving functionality absent since earlier versions.
- Tailoring enzyme interface provides EC-classified annotation with links to [[MITE Database]], enabling rapid identification of modification chemistry.
- [[KnownClusterBlast]] output simplified to confidence levels ("high," "medium," "low") for cleaner interpretation.
- Circular genome BGC detection now correctly handles clusters spanning the origin of replication, eliminating false splits.
- Adenylation domain substrate predictions now link to the [[PARAS]] predictor for improved specificity.
- JavaScript-based ClusterBlast SVG generation significantly reduces output file sizes.

## Why it matters

[[antiSMASH]] is the de facto standard for [[Genome Mining|genome mining]] of natural products; updates propagate immediately to thousands of researchers and to large-scale databases (e.g., the [[antiSMASH Database]]). The 20 new cluster types and the terpenoid/tailoring modules make previously invisible chemical space computationally accessible, directly enabling the discovery of novel bioactive compounds. Regulatory analysis improvements help connect biosynthetic potential to expression context.

## Limitations & caveats

- Fungal gene-calling was removed — users must pre-annotate fungal genomes with external tools (e.g., [[AUGUSTUS]]) before submitting, increasing the barrier for fungal genome mining.
- [[MEME]] suite support (MEME and FIMO) was deprecated due to modern system incompatibilities, disabling [[CASSIS]]-based fungal BGC border detection and [[RODEO]] scoring that relied on these tools.
- New cluster types are only as reliable as their underlying reference BGCs in [[MIBiG]]; rare or poorly characterised chemistries may still be missed.
- The tool remains annotation-dependent — incorrect gene models upstream propagate errors into cluster predictions.

## My take

This is a solid, workmanlike update that advances the field incrementally rather than fundamentally. The terpenoid module is genuinely valuable — terpenoids are the largest class of natural products yet historically underserved by automated tools. The removal of fungal gene-calling is a pragmatic but user-unfriendly decision that shifts burden to wet-lab researchers less familiar with bioinformatics pipelines. Overall, antiSMASH 8.0 consolidates its position as the essential first-pass tool for any [[Genome Mining]] project, even if deep characterisation still requires complementary tools.

## Concepts introduced
- [[Atropopeptide]] — a newly detectable RiPP class featuring axial chirality-based structural constraint, added to antiSMASH 8.0 detection rules.
- [[Tailoring Enzyme Module]] — an interface organising post-assembly-line modification enzymes by EC class with cross-links to the MITE database, enabling systematic annotation of chemical decoration steps.
- [[PARAS Predictor]] — an external adenylation domain substrate specificity predictor now linked from antiSMASH NRPS analysis outputs.

## Connects to
- [[antiSMASH]]
- [[Biosynthetic Gene Cluster]]
- [[MIBiG]]
- [[Natural Products]]
- [[NRPS]]
- [[PKS]]
- [[RiPP]]
- [[Genome Mining]]
- [[Secondary Metabolism]]

## Quotes worth keeping
> "antiSMASH keeps antiSMASH up-to-date with developments in the field and extends its overall predictive capabilities for natural product genome mining."

> Detection capacity increased from 81 to 101 detectable biosynthetic cluster types — adding archaeal RiPPs, atropopeptides, nitropropanoic acid, azoxy compounds, polyynes, deazapurines, and others.

## Citation

Blin K, Shaw S, Vader L, Szenei J, Reitz ZL, Augustijn HE, Cediel-Becerra JDD, de Crécy-Lagard V, Koetsier RA, Williams SE, Cruz-Morales P, Wongwas S, Segurado Luchsinger AE, Biermann F, Korenskaia A, Zdouc MM, Meijer D, Terlouw BR, van der Hooft JJJ, Ziemert N, Helfrich EJN, Masschelein J, Corre C, Chevrette MG, van Wezel GP, Medema MH, Weber T. antiSMASH 8.0: extended gene cluster detection capabilities and analyses of chemistry, enzymology, and regulation. *Nucleic Acids Research*. 2025;53(W1):W32–W38. https://doi.org/10.1093/nar/gkaf334

---
*Processed: 2026-05-20*
