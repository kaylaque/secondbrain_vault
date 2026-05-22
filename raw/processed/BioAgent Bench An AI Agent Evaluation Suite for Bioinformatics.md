---
title: "BioAgent Bench: An AI Agent Evaluation Suite for Bioinformatics"
source: "https://arxiv.org/html/2601.21800v2"
author:
published:
created: 2026-05-20
description:
tags:
  - "clippings"
---
Dionizije Fa <sup>∗</sup>    Marko Čuljak <sup>∗</sup>    Bruno Pandža    Mateo Čupić

###### Abstract

This paper introduces BioAgent Bench, a benchmark dataset and an evaluation suite designed for measuring the performance and robustness of AI agents in common bioinformatics tasks. The benchmark contains curated end-to-end tasks (e.g., RNA-seq, variant calling, metagenomics) with prompts that specify concrete output artifacts to support automated assessment, including stress testing under controlled perturbations. We evaluate frontier closed-source and open-weight models across multiple agent harnesses, and use an LLM-based grader to score pipeline progress and outcome validity. We find that frontier agents can complete multi-step bioinformatics pipelines without elaborate custom scaffolding, often producing the requested final artifacts reliably. However, robustness tests reveal failure modes under controlled perturbations (corrupted inputs, decoy files, and prompt bloat), indicating that correct high-level pipeline construction does not guarantee reliable step-level reasoning. Finally, because bioinformatics workflows may involve sensitive patient data, proprietary references, or unpublished IP, closed-source models can be unsuitable under strict privacy constraints; in such settings, open-weight models may be preferable despite lower completion rates. We release the dataset and evaluation suite publicly.

<table><tbody><tr><th rowspan="2"></th><td><a href="https://www.github.com/bioagent-bench/bioagent-bench">bioagent-bench/bioagent-bench</a></td></tr><tr><td><a href="https://www.github.com/bioagent-bench/bioagent-experiments">bioagent-bench/bioagent-experiments</a></td></tr></tbody></table>

Machine Learning, ICML

## 1 Introduction

![Refer to caption](https://arxiv.org/html/2601.21800v2/x1.png)

Figure 1: An overview of BioAgent Bench. Inputs to LLM agents consist of a task prompt, input data, and reference data. While solving the provided task, an agent can use general-purpose packages or specialized bioinformatics tools. After the agent finishes generation, LLM judge compares its outputs against ground truth and produces evaluation results. In addition to the standard ”vanilla” inputs, we also experiment with different perturbations to stress-test the agents. We focus our evaluation on 10 tasks in bioinformatics (each designed around a different organism, virus, or an entire ecosystem), and 10 models (5 open-weight and 5 closed-weight models).

AI agents have become a central component of developing computational workflows, including those in the life sciences. Bioinformatics is a particularly compelling domain for such agentic systems, as many routine analyses involve chaining together command-line tools, managing heterogeneous file formats, and interpreting intermediate outputs in a way that is both structured and highly subfield-specific.

Rigorously evaluating such analyses is challenging due to their complexity and the existence of a wide range of plausible outputs. Further, parameter and algorithm choices at early stages of a pipeline can drastically influence final results, thereby complicating ablation studies and weakening the robustness of drawn conclusions. These limitations make some analysis steps difficult or impossible to label with strict pass/fail criteria. Consequently, existing evaluations tend to collapse rich workflows into simplified question-answering or code-generation problems that fail to faithfully capture realistic agent behavior.

At the same time, bioinformatics pipelines can operate on highly sensitive data, such as patient tumor sequencing samples or clinical metadata, where both privacy and intellectual-property constraints prevent sharing raw datasets with third-party model providers or releasing them publicly [^28]. In these settings, practical deployment often requires models that can be run entirely within the institution’s secure environment.

To address these gaps, we introduce a benchmark dataset of common bioinformatics tasks specifically designed to evaluate large language model (LLM) agents in realistic workflows that require tool use. The benchmark covers a diverse set of tasks that reflect everyday usage scenarios for bioinformatics practitioners.

Using the benchmark, we compare the performance of frontier closed-source models with that of state-of-the-art open-source models when used as agents. We show that frontier proprietary models can successfully solve the majority of benchmark tasks without extensive custom scaffolding, suggesting that current LLM technology is already capable of supporting many routine bioinformatics workflows.

In general, our work provides three main contributions: (i) a benchmark dataset for evaluating AI agents on practical bioinformatics tasks; (ii) a systematic comparison of closed and open-source models in this setting; and (iii) an evaluation suite and an evaluation harness that executes tasks, records intermediate steps, grades outputs, and tests robustness with perturbation tests.

## 2 Related Work

#### General LLM agent benchmarks.

The evaluation of large language models as autonomous agents has emerged as a critical research direction, moving beyond traditional static benchmarks to assess models in dynamic, interactive environments. AgentBench [^12] provides a comprehensive evaluation framework spanning eight distinct environments, including operating systems, databases, knowledge graphs, and web browsing. Aiming to evaluate the abilities of LLMs to effectively utilize external tools and APIs [^11] design a benchmark comprising 73 APIs and 314 tool-use dialogues, and decompose evaluation into three progressive levels: API call accuracy, retrieval capability, and end-to-end task completion with planning. ToolBench [^22] [^21] [^6] further broadens this scope by covering 16k real-world APIs across 49 categories. Our work is most closely related to SWE-bench [^8], which focuses on software engineering and requires the models to generate patches that resolve problems reported in GitHub issues.

Compared to software engineering or general tool-use benchmarks, biomedical tasks are harder to evaluate automatically because many candidate solutions ultimately require wet-lab experiments and clinical validation. Moreover, biological systems exhibit substantial noise and heterogeneity (e.g., batch effects, protocol variation), so multiple analysis pipelines or hypotheses can be reasonable, and success is better characterized by decision quality under constraints than by a single correct output. These properties complicate benchmark construction, where the tasks must balance realism with automatic evaluation.

#### Biomedical LLM benchmarks.

A growing body of work focuses on building benchmarks for LLMs and LLM-based agents for drug discovery and biomedical data analysis tasks. For example, BioML-bench [^15] evaluates tasks where agents must parse a task description, build a pipeline, implement models, and submit predictions graded by established metrics across domains such as protein engineering, single-cell omics, biomedical imaging, and drug discovery. Broader capability-oriented evaluations include LAB-Bench [^10], a large multiple-choice benchmark covering practical biology research skills, including literature reasoning, database navigation, figure interpretation, and sequence manipulation. [^13] propose an evaluation framework to assess agent capabilities in single-cell omics analysis. Finally, BixBench [^16] focuses on data analysis in computational biology, presenting agents with real-world scenarios that require dataset exploration, multi-step analysis, and nuanced interpretation.

In contrast to benchmarks centered on data analysis and broad research text response-based capabilities, our benchmark emphasizes multi-step bioinformatics pipelines across different bioinformatics domains and computational environments.

## 3 Preliminaries

A bioinformatics pipeline is a workflow that transforms raw biological data, such as DNA/RNA sequencing reads, into analyzed outputs (e.g., alignments, expression estimates, or variant calls). Within a pipeline, it is useful to distinguish two kinds of data: input data and reference data. Input data is the sample material being processed (for example, reads from a particular experiment or individual). Reference data is the shared baseline used across many samples for steps like alignment or annotation (for example, it can be a reference genome, gene annotation, or a known variant database).

Separately, in an evaluation setting, we use a different set of terms. A task is a single instance with defined inputs and a clear success criterion. A trial is one execution of that task. A grader is the evaluation logic that scores a trial. During a trial, we record a transcript (also called a trace or trajectory), meaning the full log of what happened, including messages, tool calls, and intermediate steps. The outcome is the final produced result.

The surrounding systems are defined as follows. An evaluation harness is the system that executes tasks, captures transcripts, applies graders, and aggregates results across trials. An agent harness (or scaffold) is the system that enables a model to take actions and use tools; in this framing, an “agent” is simply the model plus that harness. An evaluation suite is a collection of related tasks designed to measure a particular capability or behavior. [^5]

## 4 Benchmark Design

BioAgent Bench comprises curated bioinformatics tasks that span common modalities and workflows: bulk and single-cell RNA-seq, comparative genomics, variant calling, metagenomics, viral metagenomics, transcript quantification, and experimental evolution. Each task is framed as an end-to-end pipeline that requires tool orchestration, file handling, and structured reporting, rather than a single-step QA. To enable automated evaluation of the results, we specify concrete output formats, typically CSV.

A common theme in both the paper and BioAgentBench is verifiability: in biology, it can be hard to tell whether an output reflects a true signal, because conclusions often depend on choices across the analysis pipeline. Bioinformatics typically follows a canonical workflow from sample preparation and measurement through preprocessing to downstream analysis and statistical inference, which BioAgentBench mirrors. In domains with small effect sizes and high sensitivity to modeling decisions, reasonable choices about design, normalization, batch correction, and statistical assumptions can yield different conclusions from the same data, making single-experiment outputs underdetermined. Examples include short-timescale evolution or selection experiments, subtle differential expression studies, cross-batch single-cell RNA-seq comparisons, and microbiome association analyses.

Taking these ideas into account, the intended purpose of the dataset selection process and curation is to make the BioAgent Bench closer to software engineer benchmarks rather than biology data analysis benchmarks. This is intended to allow for future use cases in reinforcement learning, distillation, and similar. With that in mind, the main idea was to collect tasks that are most widely used in bioinformatics and lend themselves easily to being described in code and tool-calling pipelines.

To that end, we made two constraints on the potential datasets, which drastically reduced the eligible set: (1) runtime must be below 4 hours; (2) workflows must be runnable with 48GB of RAM. By enforcing these constraints, we focus on datasets from smaller organisms where the required reference data can be provided as inputs. This comes with trade-offs: it reduces fidelity to real-world practice by excluding large-organism workflows (e.g., human sequencing) and by omitting common research tasks such as finding, downloading, and staging large reference resources. We list the tasks and their key properties in Table 1.

Table 1: BioAgent Bench tasks. ”Verifiable” indicates tasks that can be scored as a binary outcome (pass/fail)

| Identifier | Name | Language | Tool calls | Verifiable |
| --- | --- | --- | --- | --- |
| alzheimer-mouse | Alzheimer Mouse Models: Comparative Pathway Analysis | Python | ✗ | ✗ |
| comparative-genomics | Comparative Genomics: Co-evolving Gene Clusters | R | ✗ | ✗ |
| cystic-fibrosis | Cystic Fibrosis Mendelian Variant Identification | bash | ✓ | ✓ |
| deseq | RNA-Seq Differential Expression (DESeq2) | Python | ✓ | ✗ |
| evolution | Experimental Evolution Variant Calling (E. coli) | bash | ✓ | ✗ |
| giab | GIAB Variant Calling (NA12878) | bash | ✓ | ✓ |
| metagenomics | Metagenomics: Community Comparison (Cuatro Cienegas) | R | ✓ | ✗ |
| single-cell | Single-cell RNA-seq: Skeletal Muscle Exercise Response | Python | ✗ | ✗ |
| transcript-quant | Transcript Quantification (Simulated RNA-Seq) | bash | ✓ | ✓ |
| viral-metagenomics | Viral Metagenomics: Species Identification (Dolphin) | bash | ✓ | ✓ |

In BioAgent Bench, a singular task consists of: (i) a natural-language prompt that specifies the goal and expected output format (e.g. Perform metagenomic analysis of control and fertilized samples) (ii) the associated files required to execute and evaluate the task. The files include the primary input data and, when available, reference data (e.g., taxonomic database). This definition ties the instruction and data together as a single unit for agent evaluation.

## 5 Experimental Setup

To evaluate each agent, we run the model in a specific harness in a sandbox tied to a hashed run directory. As inputs to the coding agent, we provide: i) a system prompt; ii) input files; iii) prompt instructions with goal and expected output format of the outcome (see Appendix A.1–A.3). The system prompt instructs the model to generate artifacts per step (e.g., quality control, read trimming, assembly, etc.). The generated output files and the result file are passed for evaluation to the grader. The grader is an LLM - GPT-5.1, which returns several evaluation metrics for the run.

Our experimental setup and thereby results don’t compare individual LLMs directly; instead, they should be interpreted as an evaluation of agentic capabilities, i.e., the combined performance of a model and its harness.

### 5.1 Model Settings

We evaluate top-performing open and closed weights models from SWE-bench [^8] at the time of running evaluation. We run the models in three available harnesses: Claude Code [^2], Codex CLI [^17], and OpenCode [^1]. Each run uses a simple system prompt that points the LLM at the available environment, strategy for producing artifacts, and the final completion condition. Agents are run in a sandboxed folder (sandbox rules defined by each individual harness), with access to the network. Every model is run with ”high” reasoning effort if available.

### 5.2 Grader Logic

Scoring the trials is done by an LLM grader. We opted for LLM grading for multiple reasons. The first reason is that bioinformatics tasks often admit multiple valid solution paths and tool choices. To give an example, when conducting a germline variant calling analysis, starting from the same data, the agent can opt to use a GATK4 Haplotype caller pipeline [^20] which is a canonical pipeline of multiple steps, it can opt to use DeepVariant [^19] which is a deep learning model, or one of the many other germline variant calling pipelines. These pipelines result in a different number of steps needed to complete the analysis. An LLM grader lets us score the outputs and steps against a rubric without hard-coding a single canonical truth, which brings the possibility of there being a different number of steps per trial for the same task. The second reason is that bioinformatics analyses produce a large volume of intermediate files. Given the number of trials we evaluated and the nondeterminism of solution paths, a comprehensive manual review would require substantial effort from a domain expert familiar with the relevant analysis and workflow.

The grader takes as input:

- input and reference data paths
- the expected outcome (CSV/TSV table as text, ground truth)
- the agent’s outcome (CSV/TSV table as text if exists)
- agent’s trace (only folders and file paths)
- grading logic prompt (see Appendix A.4) where the grading logic prioritizes evidence of pipeline completion over numerical accuracy

The grader outputs:

- steps\_completed: number of pipeline steps the agent demonstrably completed.
- steps\_to\_completion: estimated total steps required to finish the task.
- final\_result\_reached: whether the agent produced the final requested result artifact.
- results\_match: task-specific correctness flag from rubric rules
- f1\_score: F1-score where applicable (only giab)

### 5.3 Evaluation Suite

The evaluation suite is designed not only to assess performance in a single trial but also to measure robustness across multiple trials, with and without perturbations. It supports the following settings:

- Multiple trials where we can evaluate the difference in outputs and trajectories between trials
- Trials where we test robustness to inflating the initial task prompt unnecessarily
- Trials where we test robustness to perturbations with corrupted data, and the agent is expected to recognize such data
- Trials where we test robustness to perturbations with decoy data, which the agent is not supposed to use in the pipeline

Our primary metric is completion rate (%). For each task, we evaluate whether the agent completes each required pipeline step and produces the requested final artifact in the specified format (CSV/TSV). The completion rate is the percentage of required steps that pass this check, as assessed by the LLM grader.

## 6 Results

Across BioAgent Bench tasks, frontier models achieve high pipeline completion rates. Claude Opus 4.5 attains a 100% completion rate, while Gemini 3 Pro, GPT-5.2, and Sonnet 4.5 each exceed 90%. The top results are obtained using the Codex CLI harness (see Appendix 4 for results across harnesses). Open-weight models trail on average, with the best-performing model, GLM-4.7, reaching 82.5% in the Codex CLI harness and other open-weight models ranging down to 65%.

These results suggest that current frontier models can reliably execute multi-step bioinformatics workflows end-to-end, reaching the requested final artifact without additional scaffolding. In contrast, the open-weight models evaluated here show materially lower completion rates.

To investigate whether this discrepancy is the result of a model’s internal knowledge or multi-turn agentic capabilities, we have instructed the models to produce just a higher-level plan, without execution, of the pipelines with the same input data and task prompt as in the vanilla evaluation suite. We then instructed GPT-5.1 to score each plan on a 1–5 scale. As seen in 3, planning quality correlates with overall agentic performance (Pearson r=0.61), suggesting that models with better plans generally translate into higher end-to-end pipeline completion success. However, this relationship is not deterministic; open-weight models tended to receive lower planning scores, yet some models (e.g., Gemini-Pro-3) produced weaker plans than frontier models while still completing pipelines at high success rates. This indicates that strong completion can sometimes be achieved despite comparatively lower-quality explicit planning, or that success depends on meeting a baseline level of domain knowledge, while shortcomings in agentic capabilities remain the main bottleneck for open-weight models.

We also observed a qualitative difference in failure modes during manual inspection of a subset of runs: some closed-source models appeared more prone to getting stuck in repeated error-correction loops or terminating prematurely before reaching a solution, whereas frontier models more often recovered and completed the pipeline. We did not conduct a systematic analysis of these behaviors, which we leave to future work.

![Refer to caption](https://arxiv.org/html/2601.21800v2/x2.png)

Figure 2: Model-task completion heatmap. The left panel shows a pairwise completion matrix: rows and columns correspond to models and tasks, respectively, and each cell reports the completion rate (in %) for each model and task pair. Cell color encodes the completion rate, with numeric annotations shown for readability. The right panel summarizes performance across tasks by reporting each model’s average completion rate, providing an overall ranking of models.

### 6.1 Robustness

To further systematically evaluate the agents’ understanding of the data and the underlying tasks, as a proxy for testing the biological reasoning, we evaluated the stability of results, i.e., robustness across multiple trials. For this, we used the GPT-5.2 model in Codex CLI harness <sup>1</sup>. For each task, we ran the agent for four trials and compared the stability of the results using the Jaccard Index for categorical data (e.g., KEGG Pathways, Gene IDs) and the Pearson correlation coefficient for numerical data (e.g., p-values, abundance) <sup>2</sup>. We computed the Jaccard overlap of the final results across trials of the same task as the Jaccard index:

$$
J=\frac{|I_{a}\cap I_{b}|}{|I_{a}\cup I_{b}|}
$$

For the Pearson correlation coefficient across trials within each task, we keep only IDs shared by all runs. For each value column present in all runs, we compute the Pearson correlation between the runs’ values (dropping missing rows). Then we average those correlations. If no valid correlations exist, the result is NaN.

The mean value of the overlap between categorical results, i.e., the Jaccard Index, was 0.43, while the Pearson correlation coefficient was 0.73, indicating that there was considerable variability in the final results across trials.

We hypothesized that this variability is driven by a combination of non-determinism in tool execution and between-trial differences in inferred parameters or intermediate decisions made during the pipeline (e.g., statistical inference choices). See Appendix A.7 for the isolated case study.

![Refer to caption](https://arxiv.org/html/2601.21800v2/x3.png)

Figure 3: Scatter plot comparing each model’s average plan quality score against its overall pipeline completion rate

Table 2: Jaccard index and Pearson correlation across multiple trials.

| Task | Trials | Jaccard | Pearson |
| --- | --- | --- | --- |
| alzheimer | 4 | 0.160 | 0.219 |
| comparative | 4 | 0.004 | NA |
| cystic-fibrosis | 3 | 1.000 | NA |
| deseq | 4 | 0.978 | 0.995 |
| evolution | 4 | 0.000 | NA |
| metagenomics | 4 | 0.395 | 0.746 |
| single-cell | 4 | 0.114 | 0.395 |
| transcript-quant | 4 | 1.000 | 1.000 |
| viral-metagenomics | 4 | 0.667 | 1.000 |

### 6.2 Perturbation Analysis

As a complementary component of the evaluation suite aimed at probing biological reasoning below the level of global pipeline construction, we evaluate agent behavior under task-specific perturbations. For each task and perturbation condition, we run a single trial using GPT-5.2 in the Codex CLI harness.

Table 3: Agent perturbation outcomes across tasks. Shows identified corrupt data (check is good), whether a decoy was used (cross is good), and the resulting change in completion performance due to prompt bloat (percentage points; negative values indicate degradation).

| Identifier | Corrupted | Decoy | $\Delta$ Completion (%) |
| --- | --- | --- | --- |
| alzheimer-mouse | ✗ | ✗ | \-12.5 |
| comparative-genomics | ✗ | ✓ | \-20.0 |
| cystic-fibrosis | ✗ | ✗ | 0.0 |
| deseq | ✓ | ✗ | \-100.0 |
| evolution | ✓ | ✗ | 75.0 |
| giab | ✓ | ✗ | \-20.0 |
| metagenomics | ✓ | ✓ | \-100.0 |
| single-cell | ✓ | ✗ | \-100.0 |
| transcript-quant | ✓ | ✗ | 0.0 |
| viral-metagenomics | ✓ | ✗ | 0.0 |

- Decoys: We inject decoy inputs (e.g., sequences from an unrelated organism) to test whether the agent can correctly contextualize the data and exclude irrelevant files from downstream analysis.
- Corrupted inputs: We synthetically corrupt selected input files and measure whether the agent detects the corruption and avoids proceeding with invalid data.
- Prompt bloat: We augment the task prompt with additional, topically related but non-essential text to assess robustness to distraction and instruction overload.

To characterize the agent’s response to these perturbations, we manually inspected execution traces. For the decoy condition, we retrieved all code blocks and tool invocations that referenced the decoy filenames and verified whether the decoy data were incorporated into analysis (e.g., used in commands, loaded into scripts, or included in downstream result aggregation). For the corrupted-input condition, we inspected agent messages and tool calls that referenced the corrupted filenames to determine whether the agent explicitly flagged the issue and whether it attempted to continue the pipeline regardless.

As summarized in Table 3, the agent correctly identified corrupted inputs in 7/10 tasks. Decoy files were used erroneously in 2/10 tasks. Prompt bloat had a pronounced negative effect on overall completion: across tasks and trials, agents completed 28% fewer steps relative to the unperturbed setting. Task-specific perturbations are defined in the Appendix A.8

#### Failure modes under corrupted inputs.

Manual inspection of runs under the corrupted-input condition revealed several distinct failure modes. In some tasks, the agent failed to detect corruption and proceeded as if the inputs were valid. For example, in alzheimer-mouse, the agent continued with differential-expression analysis despite an obviously corrupted DESeq2-style distribution. In comparative-genomics, it again operated indiscriminately over available sequences rather than validating input integrity.

In other tasks, the agent *did* flag corruption but continued the pipeline regardless, typically by attempting to “route around” the issue via alternative analyses or reference downloads. This pattern was observed in evolution (corruption detected, but the agent proceeded with downstream analysis after downloading references), as well as in giab and metagenomics (corruption detected, yet analysis continued). A related variant occurred in transcript-quant: the agent identified the corrupted input but still attempted subsequent steps until a downstream tool invocation failed due to missing or unusable outputs.

Finally, we observed cases where corruption led to early termination, either appropriately or due to cascading failures. In deseq, the input FASTA sequences were unusable, and the agent exited early. In contrast, single-cell represents the desired behavior: the agent identified the corrupted data and did not proceed with further analysis. The cystic-fibrosis task exhibited a different but practically similar failure mode: the metadata were scrambled and inconsistent with the prompt specification, and the agent continued the analysis, yielding incorrect results.

We note that these behaviors are also sensitive to the type and severity of corruption introduced in our ablation. Some perturbations rendered files effectively unreadable, in which case downstream tool failures can force early termination irrespective of the agent’s intent. Other perturbations preserved superficially valid structure (e.g., files that load or parse but contain implausible values or scrambled semantics), allowing pipelines to run to completion despite being scientifically invalid. Importantly, while the latter category may not trigger hard errors, it would typically be readily apparent to a human practitioner that the inputs are not trustworthy and should not be used for inference.

#### Failure modes with decoy files

A closer inspection of the decoy failures suggests two recurring patterns. First, in the comparative genomics task, file selection was implemented via a shallow filename heuristic: the agent globbed all inputs matching the.genomic.fna suffix and therefore unintentionally included the decoy organism alongside the intended genome. Second, in the metagenomics task, the agent selected an inappropriate reference database, using a viral database in place of the bacterial database required by the task. Both cases are consistent with a failure to ground tool configuration in a biological context, instead relying on surface-level cues (filenames and readily available defaults) that are insufficient under adversarial or confounded inputs. In all other cases, the agent correctly discarded the decoy files.

#### Failure modes with prompt bloat

Prompt bloat induced a distinct set of behavioral failures, most clearly visible in tasks with complete degradation ($-100\%$ completion). In these cases, the agent exhibited the same qualitative symptoms we observed for weaker (open-weight) models: rather than making incremental progress through tool use and stateful execution, the agent repeatedly restated the task, cycled through superficial reformulations of the instructions, and then terminated early without producing substantive intermediate artifacts.

## 7 Discussion

BioAgent Bench aims to evaluate agentic bioinformatics behavior as it occurs in practice: selecting the right inputs and references, orchestrating multi-step tool calls, writing analysis scripts, conducting statistical inference, and delivering a concrete final result. Our results suggest that today’s frontier agents can often execute canonical workflows end-to-end with minimal scaffolding, but that pipeline completion can substantially overestimate reliability.

The observed variability between runs (e.g., different Salmon flags, differing statistical choices) reflects two realities: (i) non-determinism in agent decisions and tooling, and (ii) genuine degrees of freedom in analysis pipelines. Stability metrics (Jaccard overlap, Pearson correlation) quantify how sensitive an agent is to small internal decision differences and are an important part of the overall evaluation of agent performance and behavior.

In practice, we expect useful agents to become more stable when given explicit constraints (fixed tool versions, parameter templates, prespecified reference datasets) and when equipped with internal policies such as “prefer defaults unless justified by data-driven checks.” BioAgent Bench can measure how well different harnesses and prompting strategies enforce this discipline.

Across tasks, high completion rates indicate that agents can: interpret the intended goal, choose the correct tools, manage environments (mamba, R packages), and recover from many routine tool errors. This supports the view that state-of-the-art agents can act as effective workflow assistants for common bioinformatics analyses without much custom engineering.

Open-weight models are beginning to close the gap, but remain behind frontier closed models in this evaluation. In our runs, the strongest open-weight models often produced workable pipelines and sometimes achieved competitive completion, yet exhibited greater variability and more frequent failures to reach the requested final artifact. One plausible explanation is that the deficit is not purely “bioinformatics knowledge,” but a combination of (i) weaker end-to-end agentic competence (tool use reliability, error recovery, and state tracking over long executions) and (ii) reduced ability to form and maintain high-quality execution plans. Consistent with this, we observe a positive correlation between plan ratings and completion, suggesting that explicit planning quality is a meaningful, though not sufficient, predictor of downstream success. Importantly, the relationship is not deterministic because some models can complete tasks despite producing weaker explicit plans, indicating that agentic capabilities can compensate.

However, our robustness experiments highlight a key gap: correct higher-level pipeline construction does not imply correct step-level reasoning. A practical implication is that benchmarks (and deployments) should treat pipeline completion as a necessary but insufficient criterion. For sensitive use cases such as in clinical diagnostics, the relevant question is not “does it produce a result?” but “can it reliably detect when it should not proceed, and can it justify choices with evidence grounded in the data and context?”

Despite lower current completion rates, open-weight models play an important role in settings where privacy is crucial. Many realistic workflows involve sensitive patient-derived sequencing data for cancer screening, proprietary reference collections, or unpublished IP where routing data to model providers may be unacceptable. In such contexts, locally deployable agents can be preferable even when they underperform on aggregate benchmarks, because they enable stronger governance and reduce organizational risk. From this perspective, improving open-weight agent performance is not only a matter of competitiveness but an enabling step for safe and compliant deployment of agentic systems in biomedical research and clinical environments.

## 8 Limitations

#### LLM grading can be subjective and biased.

LLM graders enable scalable evaluation when multiple solution paths are valid, but scores can depend on rubric wording, trace verbosity, and how intermediate artifacts are presented. This can reward trials that look plausible even when step-level reasoning is wrong, and it can also introduce run-to-run inconsistency (e.g., differing numbers of accepted steps), which already occurs in the current suite.

Benchmark constraints limit real-world fidelity. Resource caps (runtime $<4$ hours, $\leq 48$ GB RAM) improve reproducibility but narrow the task and dataset space. They tend to favor smaller organisms and well-packaged inputs, underrepresenting common failure modes such as large genomes, heterogeneous cohorts, messy metadata, and long-running pipelines. The benchmark also excludes a major part of applied bioinformatics—discovering, curating, and justifying external references and best practices from primary sources.

Robustness testing is limited in scope and sampling. Perturbation analysis uses a single trial per task and condition, limiting uncertainty estimates and variance comparisons across models and harnesses; results should be treated as suggestive. Perturbation coverage is also narrow relative to practice, where confounders include multiple plausible files, misleading names/metadata, partial truncations, and subtle format violations. Corruption difficulty depends on construction: some cases are obvious (malformed files), while more realistic and safety-relevant cases remain syntactically valid but biologically implausible.

## 9 Conclusion

BioAgent Bench provides an end-to-end benchmark and an evaluation suite for bioinformatics agents, capturing realistic workflows that require tool orchestration, artifact production, and structured outputs. Frontier agents complete canonical pipelines with high success rates without heavy scaffolding, but robustness tests show that it comes with brittle step-level behavior such as shallow file selection heuristics, weak input validation, and sensitivity to distraction. By making these failure modes measurable, BioAgent Bench shifts evaluation from “can it finish?” to “can it finish reliably, for the right reasons?”

Next, we will expand task and dataset diversity (including larger and messier inputs), add tasks that require sourcing and justifying external references, and strengthen robustness evaluation with richer perturbations and automated scoring that integrates robustness into the primary metrics.

## Impact Statement

This paper presents BioAgent Bench, a benchmark dataset and evaluation suite for end-to-end bioinformatics agent workflows. A central intended impact is to improve agent evaluation for practical use cases. A second intended impact is to support the development of private, locally deployable agentic systems using open-weight models, by enabling standardized benchmarking to track the capabilities gap between open-source and the closed frontier. We also expect BioAgent Bench to be useful as a target for improving agentic behavior via methods such as fine-tuning, distillation, and reinforcement learning on verifiable multi-step tasks.

## Acknowledgments

This work was funded by the European Union – NextGenerationEU, project NPOO.C3.2.R2-I1.04. Marko Čuljak was supported by the Croatian Science Foundation (HRZZ) Young Researchers’ Career Development Project, grant DOK-NPOO-2023-10-1392.

## References

## Appendix A Appendix

### A.1 System prompt

<svg id="A1.SS1.p1.pic1" height="515.78" overflow="visible" version="1.1" viewBox="0 0 600 515.78" width="600"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" transform="translate(0,515.78) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g style="--ltx-fill-color:#A6A6A6;" fill="#A6A6A6" fill-opacity="1.0"><path style="stroke:none" d="M 0 12.36 L 0 503.41 C 0 510.24 5.54 515.78 12.36 515.78 L 587.64 515.78 C 594.46 515.78 600 510.24 600 503.41 L 600 12.36 C 600 5.54 594.46 0 587.64 0 L 12.36 0 C 5.54 0 0 5.54 0 12.36 Z"></path></g><g style="--ltx-fill-color:#FBFBFB;" fill="#FBFBFB" fill-opacity="1.0"><path style="stroke:none" d="M 0.55 12.36 L 0.55 503.41 C 0.55 509.94 5.84 515.22 12.36 515.22 L 587.64 515.22 C 594.16 515.22 599.45 509.94 599.45 503.41 L 599.45 12.36 C 599.45 5.84 594.16 0.55 587.64 0.55 L 12.36 0.55 C 5.84 0.55 0.55 5.84 0.55 12.36 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 28.11 23.01)"><foreignObject style="--ltx-fg-color:#000000;--ltx-fo-width:39.3em;--ltx-fo-height:34.15em;--ltx-fo-depth:0.2em;" width="543.78" height="475.3" transform="matrix(1 0 0 -1 0 472.53)" overflow="visible" color="#000000"><span id="A1.SS1.p1.pic1.1.1.1.1.1" style="width:41.59em;"><span id="A1.SS1.p1.pic1.1.1.1.1.1.1"><span id="A1.SS1.p1.pic1.1.1.1.1.1.1.1" style="font-size:90%;">You are an expert bioinformatics agent that assists users with bioinformatics and computational biology tasks. You are an expert in genomics, transcriptomics, proteomics and related -omics domains, and you follow best practices from the field. Environment Management You are already working inside the mamba environment named {env_name}. Never attempt to activate the base environment; keep using {env_name} unless instructed to create a new one. Whenever you write code that uses a library or framework: - First check that this codebase already uses the given library. - Use the ’mamba list’ command to check if the library is installed. - Install any missing libraries or packages using mamba. - Always install packages into the bioinformatics environment.</span></span> <span id="A1.SS1.p1.pic1.1.1.1.1.1.2"><span id="A1.SS1.p1.pic1.1.1.1.1.1.2.1" style="font-size:90%;">Code Style For making network requests or large loops add a progress meter. Do not keep downloaded data in memory; export it to files so you don’t have to download again. In case of errors, wrap subprocess calls in try/except blocks and output the exception error so you know exactly why the subprocess call failed. Example: result = subprocess.run([’ls’, ’-l’], capture_output=True, text=True) except subprocess.CalledProcessError as e: print(f’Error: e.stderr’) When working with file paths, always use absolute paths rather than relative.</span></span> <span id="A1.SS1.p1.pic1.1.1.1.1.1.3"><span id="A1.SS1.p1.pic1.1.1.1.1.1.3.1" style="font-size:90%;">Task execution If inputs are missing/ambiguous, try to derive them using other tools. Write outputs into stage-scoped directories under special directory ’outputs/’. Example: ’outputs/0_trimming/’ ’outputs/1_alignment/’ ’outputs/3_postprocessing/’ Before starting the task run the ’tree’ command to see which files have been generated. If later you are running the tree command you should ignore.snakemake directories. Example: tree -I ’.snakemake’ Don’t just use the integers for enumerating the steps; also describe the steps. Example: ’0_processing’</span></span> <span id="A1.SS1.p1.pic1.1.1.1.1.1.4"><span id="A1.SS1.p1.pic1.1.1.1.1.1.4.1" style="font-size:90%;">Finalizing the pipeline Output the final results in the same format as was asked by the user in the provided example. Output the final results into a results/ directory. Once the final result has been generated and placed into the results/ directory, you should stop executing the task.</span></span></span></foreignObject></g></g></svg>

### A.2 Task prompts

- alzheimer-mouse: Perform a comparative differential expression analysis of three different Alzheimer’s Disease mouse models (5xFAD, 3xTG-AD, and PS3O1S) to identify shared molecular KEGG pathways. The output should be a CSV file with the following columns: pathway, 5xFAD\_pvalue, 3xTG\_AD\_pvalue, PS3O1S\_pvalue.
- comparative-genomics: Reconstruct phylogeny and identify COGs across four Micrococcus genomes; filter clusters present in all genomes, coding-only, with high-confidence annotations. The output should be a CSV file with the following columns: cluster\_number, consensus\_annotation.
- cystic-fibrosis: Find the genetic cause of Cystic fibrosis; identify the causal recessive variant consistent with affected siblings NA12885, NA12886, and NA12879. The output should be a CSV file with the following columns: chromosome, position, variant\_id, reference, alternate, gene\_name, gene\_id, annotation, impact, transcript\_id, hgvs\_c, hgvs\_p, clinical\_significance, diseases, review\_status, rs\_id.
- deseq: Identify differentially expressed genes between planktonic and biofilm conditions of Candida parapsilosis. The output should be a CSV file with the following columns: gene\_id, log2FoldChange, pvalue, padj.
- evolution: Identify and annotate genome variants in two evolved lines relative to an ancestor lines of E. coli; report only variants shared by both evolved lines of moderate or higher predicted severity. The output should be a CSV file with the following columns: chrom, pos, ref, alt, gene, impact, effect, status.
- metagenomics: Perform metagenomic analysis of control (JC1A) and fertilized (JP4D) samples to classify microbial taxa and estimate relative abundances. The output should be a CSV file with the following columns: OTU, Kingdom, Phylum, JP4D, JC1A.
- single-cell: Analyze single-cell RNA-seq data from pre- and post-exercise skeletal muscle samples. Perform clustering, cell type identification, and differential expression analysis within each cell type between conditions. The output should be a CSV file with the following columns: cluster\_id, predicted\_cell\_type, gene\_name, logfoldchanges, pvals, pvals\_adj, direction, abs\_logfc.
- transcript-quant: Perform transcript quantification on the provided paired-end RNA-Seq reads using the transcriptome reference. The output should be a.tsv file with the following columns: transcript\_id, count.
- viral-metagenomics: Analyze paired-end metagenomic sequencing data from a dolphin fecal sample to identify potential viral agents. Assemble and classify contigs, then summarize results by taxonomic domain and species. The output should be a CSV file with the following columns: contig\_count, domain, species.

### A.3 Task description

- alzheimer-mouse [^18]: Alzheimer Mouse Models: Comparative Pathway Analysis. Analyze 5xFAD, 3xTG-AD, and PS301S mouse models: normalize counts, perform differential expression, run KEGG pathway enrichment, and compare shared pathways across models.
- comparative-genomics [^9]: Comparative Genomics: Co-evolving Gene Clusters. The datasets consists FASTA sequences and GFF annotations of a microbial genome for Micrococcus. The goal of is to do phylogenetic reconstruction of clusters of orthologous co-evolving genes; identify functionally conserved gene clusters across the genomes and group them into co-evolving functional modules.
- cystic-fibrosis [^3]: Cystic Fibrosis Mendelian Variant Identification. The sample dataset is a simulated dataset for finding the genetic cause of Cystic fibrosis. The dataset is real sequencing data from CEPH\_1463 dataset provided by the Complete Genomics Diversity Panel. It consists of sequencing of a family: 4 grandparents, 2 parents and 11 siblings. A known Mandelian disease mutation has been added on three siblings, taking care to be consistent with the underlying haplotype structure. The goal is to find the mutation causing the Mendelian recessive trait - Cystic Fibrosis.
- deseq [^7]: RNA-Seq Differential Expression (DESeq2). The dataset consists of RNA-Seq samples from Candida parapsilosis wild-type (WT) strains grown in planktonic and biofilm conditions, generated as part of a study on gene expression and biofilm formation. The samples were sequenced on the Illumina HiSeq 2000 platform. The goal of this analysis is to perform differential expression analysis using DESeq2 to identify genes that are significantly up- or down-regulated between planktonic and biofilm conditions, providing insights into biofilm-associated transcriptional changes.
- evolution [^24]: Experimental Evolution Variant Calling (E. coli). The experiment follows a similar strategy as in what is called an experimental evolution experiment. The final aim is to identify the genome variations in evolved lines of E. coli. The data is composed of a single ancestor line and two evolved lines. The data is from a paired-end sequencing run data from an Illumina HiSeq. This data has been post-processed in two ways already. All sequences that were identified as belonging to the PhiX genome have been removed. Illumina adapters have been removed as well already.
- giab [^30]: GIAB Variant Calling. The GIAB dataset consists of Agilent SureSelect v7 exome sequencing ($\sim$ 75M reads) from the NA12878 reference sample, providing high-confidence benchmark variants on GRCh38. The task is to perform germline variant calling on NA12878.
- metagenomics [^29]: Metagenomics: Community Comparison (Cuatro Ciénegas). The metagenomics dataset consists of sequencing reads from the Cuatro Ciénegas Basin, comparing microbial communities under control (JC1A) and nutrient-enriched (JP4D) conditions. The task is to profile taxonomic composition and report relative abundances of bacterial taxa.
- single-cell [^14]: Single-cell RNA-seq: Skeletal Muscle Exercise Response. The single-cell RNA-seq dataset consists of human skeletal muscle samples collected before and after acute exercise, sequenced with 10X Genomics. The task is to identify cell types and determine their transcriptional responses to exercise.
- transcript-quant [^27]: Transcript Quantification (Simulated RNA-Seq). The goal is to quantify transcript expression levels from paired-end RNA-Seq reads (reads\_1.fq.gz, reads\_2.fq.gz) using the provided reference transcriptome (transcriptome.fa). Because the data is simulated, the quantification should exactly reproduce the underlying counts. The results represent a mapping from transcript IDs $\rightarrow$ read counts.
- viral-metagenomics [^4]: Viral Metagenomics: Species Identification (Dolphin). The viral metagenomics dataset consists of paired-end sequencing reads from a dolphin with gastroenteritis of suspected viral origin. The task is to identify viral species present in the fecal sample by assembling and classifying contigs.

### A.4 Grading logic prompt

<svg id="A1.SS4.p1.pic1" height="1857.69" overflow="visible" version="1.1" viewBox="0 0 600 1857.69" width="600"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" transform="translate(0,1857.69) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g style="--ltx-fill-color:#A6A6A6;" fill="#A6A6A6" fill-opacity="1.0"><path style="stroke:none" d="M 0 12.36 L 0 1845.33 C 0 1852.15 5.54 1857.69 12.36 1857.69 L 587.64 1857.69 C 594.46 1857.69 600 1852.15 600 1845.33 L 600 12.36 C 600 5.54 594.46 0 587.64 0 L 12.36 0 C 5.54 0 0 5.54 0 12.36 Z"></path></g><g style="--ltx-fill-color:#FBFBFB;" fill="#FBFBFB" fill-opacity="1.0"><path style="stroke:none" d="M 0.55 12.36 L 0.55 1845.33 C 0.55 1851.85 5.84 1857.14 12.36 1857.14 L 587.64 1857.14 C 594.16 1857.14 599.45 1851.85 599.45 1845.33 L 599.45 12.36 C 599.45 5.84 594.16 0.55 587.64 0.55 L 12.36 0.55 C 5.84 0.55 0.55 5.84 0.55 12.36 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 28.11 20.24)"><foreignObject style="--ltx-fg-color:#000000;--ltx-fo-width:39.3em;--ltx-fo-height:131.33em;--ltx-fo-depth:0em;" width="543.78" height="1817.21" transform="matrix(1 0 0 -1 0 1817.21)" overflow="visible" color="#000000"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1" style="width:41.59em;"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.1"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.1.1" style="font-size:90%;">You are a strict, impartial Bioinformatics Pipeline Judge. Your job is to evaluate an LLM agent’s work for executing a bioinformatics pipeline instructed by the prompt. The LLM agent was given an instruction to output each processing step in a separate folder. The data to evaluate each agent is given as follows:</span></span> <span id="A1.I3"><span id="A1.I3.i1" style="list-style-type:none;">1. <span id="A1.I3.i1.p1"><span id="A1.I3.i1.p1.1"><span id="A1.I3.i1.p1.1.1" style="font-size:90%;">You are given the paths of the input and the reference data which the agent was given to work with.</span></span></span></span> <span id="A1.I3.i2" style="list-style-type:none;">2. <span id="A1.I3.i2.p1"><span id="A1.I3.i2.p1.1"><span id="A1.I3.i2.p1.1.1" style="font-size:90%;">You are given the whole directory structure of the agent’s work and it is your job to estimate how close to completing the pipeline the agent came.</span></span></span></span> <span id="A1.I3.i3" style="list-style-type:none;">3. <span id="A1.I3.i3.p1"><span id="A1.I3.i3.p1.1"><span id="A1.I3.i3.p1.1.1" style="font-size:90%;">You are given the final results which the agent was instructed to produce, if they exist</span></span> </span></span><span id="A1.I3.i4" style="list-style-type:none;">4. <span id="A1.I3.i4.p1"><span id="A1.I3.i4.p1.1"><span id="A1.I3.i4.p1.1.1" style="font-size:90%;">You are givne the truth data which is the expected output of the prompted pipeline.</span></span></span></span> <span id="A1.I3.i5" style="list-style-type:none;">5. <span id="A1.I3.i5.p1"><span id="A1.I3.i5.p1.1"><span id="A1.I3.i5.p1.1.1" style="font-size:90%;">You are given the prompt which the agent was given to complete.</span></span></span></span></span> <span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.2"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.2.1" style="font-size:90%;">Inputs (provided at evaluation time)</span></span> <span id="A1.I4"><span id="A1.I4.i1" style="list-style-type:none;">• <span id="A1.I4.i1.p1"><span id="A1.I4.i1.p1.1"><span id="A1.I4.i1.p1.1.1" style="font-size:90%;">1. Input data: {input_data}</span></span> </span></span><span id="A1.I4.i2" style="list-style-type:none;">• <span id="A1.I4.i2.p1"><span id="A1.I4.i2.p1.1"><span id="A1.I4.i2.p1.1.1" style="font-size:90%;">2. Reference data: {reference_data}</span></span> </span></span><span id="A1.I4.i3" style="list-style-type:none;">• <span id="A1.I4.i3.p1"><span id="A1.I4.i3.p1.1"><span id="A1.I4.i3.p1.1.1" style="font-size:90%;">3. Processing tree: {processing_tree}</span></span> </span></span><span id="A1.I4.i4" style="list-style-type:none;">• <span id="A1.I4.i4.p1"><span id="A1.I4.i4.p1.1"><span id="A1.I4.i4.p1.1.1" style="font-size:90%;">4. Results: {results}</span></span> </span></span><span id="A1.I4.i5" style="list-style-type:none;">• <span id="A1.I4.i5.p1"><span id="A1.I4.i5.p1.1"><span id="A1.I4.i5.p1.1.1" style="font-size:90%;">5. Truth: {truth}</span></span> </span></span><span id="A1.I4.i6" style="list-style-type:none;">• <span id="A1.I4.i6.p1"><span id="A1.I4.i6.p1.1"><span id="A1.I4.i6.p1.1.1" style="font-size:90%;">6. Prompt: {task_prompt}</span></span> </span></span></span><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.3"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.3.1" style="font-size:90%;">Evaluation rules:</span></span> <span id="A1.I5"><span id="A1.I5.i1" style="list-style-type:none;">• <span id="A1.I5.i1.p1"><span id="A1.I5.i1.p1.1"><span id="A1.I5.i1.p1.1.1" style="font-size:90%;">Priortize evaluation of the pipeline completion over the correctness of the results.</span></span></span></span> <span id="A1.I5.i2" style="list-style-type:none;">• <span id="A1.I5.i2.p1"><span id="A1.I5.i2.p1.1"><span id="A1.I5.i2.p1.1.1" style="font-size:90%;">If gene names are of different naming conventions, the result is still considered valid.</span></span></span></span> <span id="A1.I5.i3" style="list-style-type:none;">• <span id="A1.I5.i3.p1"><span id="A1.I5.i3.p1.1"><span id="A1.I5.i3.p1.1.1" style="font-size:90%;">For estimating the number of steps to completion, try to estimate which bioinformatic-relevant steps are should be completed.</span></span></span></span> <span id="A1.I5.i4" style="list-style-type:none;">• <span id="A1.I5.i4.p1"><span id="A1.I5.i4.p1.1"><span id="A1.I5.i4.p1.1.1" style="font-size:90%;">Count upstream steps only if their expected artifacts are present (e.g., MultiQC, count matrix, indexing files).</span></span></span></span> <span id="A1.I5.i5" style="list-style-type:none;">• <span id="A1.I5.i5.p1"><span id="A1.I5.i5.p1.1"><span id="A1.I5.i5.p1.1.1" style="font-size:90%;">Don’t count placeholders or mock completion as a completed steps.</span></span></span></span> <span id="A1.I5.i6" style="list-style-type:none;">• <span id="A1.I5.i6.p1"><span id="A1.I5.i6.p1.1"><span id="A1.I5.i6.p1.1.1" style="font-size:90%;">For example think about p-values, logfold values, or other statistics if present.</span></span></span></span> <span id="A1.I5.i7" style="list-style-type:none;">• <span id="A1.I5.i7.p1"><span id="A1.I5.i7.p1.1"><span id="A1.I5.i7.p1.1.1" style="font-size:90%;">To be sure that there’s no mocking or hallucinated values, make sure that prior steps have been generated.</span></span></span></span> <span id="A1.I5.i8" style="list-style-type:none;">• <span id="A1.I5.i8.p1"><span id="A1.I5.i8.p1.1"><span id="A1.I5.i8.p1.1.1" style="font-size:90%;">{results_match_guidance}</span></span> </span></span></span><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.4"><span id="A1.SS4.p1.pic1.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.2.1.p2.4.1" style="font-size:90%;">Metrics to return</span></span> <span id="A1.I6"><span id="A1.I6.i1" style="list-style-type:none;">1. <span id="A1.I6.i1.p1"><span id="A1.I6.i1.p1.1"><span id="A1.I6.i1.p1.1.1" style="font-size:90%;">steps_completed: int --- The number of steps that the agent completed.</span></span></span></span> <span id="A1.I6.i2" style="list-style-type:none;">2. <span id="A1.I6.i2.p1"><span id="A1.I6.i2.p1.1"><span id="A1.I6.i2.p1.1.1" style="font-size:90%;">steps_to_completion: int --- The number of steps that the agent was expected to complete.</span></span></span></span> <span id="A1.I6.i3" style="list-style-type:none;">3. <span id="A1.I6.i3.p1"><span id="A1.I6.i3.p1.1"><span id="A1.I6.i3.p1.1.1" style="font-size:90%;">final_result_reached: bool --- Whether the agent reached the final result.</span></span></span></span> <span id="A1.I6.i4" style="list-style-type:none;">4. <span id="A1.I6.i4.p1"><span id="A1.I6.i4.p1.1"><span id="A1.I6.i4.p1.1.1" style="font-size:90%;">notes: str --- Summarize where the agent stopped if stopped and what steps are left to be done.</span></span></span></span> <span id="A1.I6.i5" style="list-style-type:none;">5. <span id="A1.I6.i5.p1"><span id="A1.I6.i5.p1.1"><span id="A1.I6.i5.p1.1.1" style="font-size:90%;">results_match: bool --- Set to true/false (or 1/0) per the rule above.</span></span></span></span></span></span> <span id="A1.SS4.p1.pic1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.p1"><span id="A1.SS4.p1.pic1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.p1.1"><span id="A1.SS4.p1.pic1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.p1.1.1" style="font-size:90%;">You are supposed to return the metrics as a JSON object with fields that satisifes the schema: EvaluationResults</span></span></span></span></foreignObject></g></g></svg>

### A.5 Harness evaluation

Table 4 shows completions rates per model and task for different harnesses.

Table 4: Model completion rates by harness.

| Model | codex-cli | claude-code | opencode |
| --- | --- | --- | --- |
| claude-opus-4-5 | 100.00 | 96.00 | 93.33 |
| claude-sonnet-4-5 | 92.50 | 90.00 | 83.33 |
| gpt-5-1 | 38.50 |  |  |
| gpt-5-1-codex | 74.67 |  |  |
| gpt-5-1-codex-max | 81.67 |  |  |
| gpt-5-2 | 92.50 |  | 87.50 |
| devstral-2512 | 47.92 |  | 65.00 |
| gemini-3-pro-preview | 96.57 |  | 67.98 |
| glm-4-7 | 82.50 |  | 75.00 |
| kimi-k2-thinking | 65.67 |  | 80.50 |
| minimax-m2.1 | 12.50 |  | 73.58 |
| qwen3-coder | 0.00 |  | 62.92 |

### A.6 Robustness metrics

Categorical and numerical values used for calculation of Jaccard index and Pearson correlation for evaluating robustness across multiple trials of the same task.

Table 5: Columns used to compute Jaccard overlap (IDs) and Pearson correlation (values) for each task

| Task | Jaccard columns (IDs) | Pearson columns (values) |
| --- | --- | --- |
| alzheimer | pathway | 3xTG\_AD\_pvalue, 5xFAD\_pvalue, PS3O1S\_pvalue |
| comparative | consensus\_annotation | none (Pearson not computed) |
| cystic-fibrosis | chromosome, position, reference, alternate | none (Pearson not computed) |
| deseq | gene\_id | baseMean, log2FoldChange, padj, pvalue |
| evolution | chrom, pos, ref, alt | none (Pearson not computed) |
| metagenomics | Phylum | JC1A, JP4D |
| single-cell | predicted\_cell\_type, gene\_name | abs\_logfc, logfoldchanges, pvals, pvals\_adj |
| transcript-quant | transcript\_id | count |
| viral-metagenomics | domain, species | contig\_count |

### A.7 Robustness case study

To probe potential sources of variability, in the simplest task - transcript-quant we manually compared four trial logs;: otlp-04c96955, otlp-14905b2e, otlp-bfbd6a48, otlp-f1d8fc0b. Across all trials, the high-level workflow is consistent and correct: Salmon builds an index from the transcriptome and then quantifies directly from paired FASTQ inputs.

The main differences were the optional flags enabled within Salmon:

- otlp-04c96955 and otlp-14905b2e: baseline runs using --validateMappings without bias-correction flags.
- otlp-bfbd6a48: enables --keepDuplicates during indexing; quantification remains baseline.
- otlp-f1d8fc0b: enables --gcBias and --seqBias during quantification (in addition to --validateMappings).

In summary, two runs are baseline, one differs at indexing, and one enables bias correction at quantification, which shows that there was no consistency across trials. These configuration differences provide a plausible mechanism for between-trial variability in downstream estimates.

### A.8 Perturbation settings

We evaluated robustness with three perturbation settings: prompt bloat, corrupt inputs, and decoy inputs.

#### Prompt bloat.

We generated long, task-specific but irrelevant background text. During perturbation tests, the corresponding block is prepended to the task prompt, increasing token count without adding task-critical instructions. The additional word counts per task are:

| Task | Additional words |
| --- | --- |
| alzheimer\_mouse | 1271 |
| comparative\_genomics | 1877 |
| cystic\_fibrosis | 1452 |
| deseq | 832 |
| evolution | 1070 |
| giab | 1066 |
| metagenomics | 871 |
| single\_cell | 958 |
| transcript\_quant | 808 |
| viral\_metagenomics | 1362 |

An example of a bloated prompt for the metagenomics task:

<svg id="A1.SS8.SSS0.Px1.p4.pic1" height="1329.39" overflow="visible" version="1.1" viewBox="0 0 600 1329.39" width="600"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" transform="translate(0,1329.39) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g style="--ltx-fill-color:#A6A6A6;" fill="#A6A6A6" fill-opacity="1.0"><path style="stroke:none" d="M 0 12.36 L 0 1317.03 C 0 1323.86 5.54 1329.39 12.36 1329.39 L 587.64 1329.39 C 594.46 1329.39 600 1323.86 600 1317.03 L 600 12.36 C 600 5.54 594.46 0 587.64 0 L 12.36 0 C 5.54 0 0 5.54 0 12.36 Z"></path></g><g style="--ltx-fill-color:#FBFBFB;" fill="#FBFBFB" fill-opacity="1.0"><path style="stroke:none" d="M 0.55 12.36 L 0.55 1317.03 C 0.55 1323.55 5.84 1328.84 12.36 1328.84 L 587.64 1328.84 C 594.16 1328.84 599.45 1323.55 599.45 1317.03 L 599.45 12.36 C 599.45 5.84 594.16 0.55 587.64 0.55 L 12.36 0.55 C 5.84 0.55 0.55 5.84 0.55 12.36 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 28.11 23.01)"><foreignObject style="--ltx-fg-color:#000000;--ltx-fo-width:39.3em;--ltx-fo-height:92.95em;--ltx-fo-depth:0.2em;" width="543.78" height="1288.92" transform="matrix(1 0 0 -1 0 1286.15)" overflow="visible" color="#000000"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1" style="width:41.59em;"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.1"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.1.1" style="font-size:90%;">The metagenomics dataset consists of sequencing reads derived from environmental samples collected in the Cuatro Ciénegas Basin, a uniquely structured aquatic and sedimentary ecosystem known for hosting diverse microbial life across fine-scale nutrient gradients. In this task, you are comparing microbial communities under two conditions: a control condition (JC1A) and a nutrient-enriched condition (JP4D). While both sets of reads originate from the same broader ecological setting, the key idea is that nutrient enrichment can shift which microbes thrive, which decline, and how the overall community composition redistributes across taxonomic groups.</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.2"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.2.1" style="font-size:90%;">Metagenomics, in the broad conceptual sense, is the study of genetic material recovered directly from a mixture of organisms present in a sample. Unlike approaches that focus on a single isolated organism, metagenomics aims to capture a ‘‘community snapshot’’ of many microbes at once---often bacteria, but potentially also archaea, viruses, and microbial eukaryotes depending on the sample and what is detectable. In practice, the raw material you start with is a large collection of short fragments of biological sequence information (reads) that collectively reflect the organisms whose DNA (or genetic material) was present in the sampled environment. Because environmental samples contain many organisms simultaneously, the reads are interleaved: fragments from different taxa are mixed together, and the analysis goal is to infer ‘‘who is there’’ and ‘‘in what proportions,’’ rather than reconstructing one single genome.</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.3"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.3.1" style="font-size:90%;">A central concept in community metagenomics is taxonomic composition, which refers to the list of taxa detected and their relative representation. A taxon is a named biological grouping such as a species, genus, family, order, class, or phylum. Depending on the data and how confidently reads can be attributed, taxonomic assignments may be made at different ranks. It is common in microbial community summaries to report abundances at a rank that is both informative and reasonably stable---often genus or family---though the appropriate rank can vary based on the confidence and granularity of classification. Importantly, taxa are not merely labels: they are a structured hierarchy, meaning that two distinct species may belong to the same genus, and multiple genera may belong to the same family, and so on. When comparing communities, shifts may be visible at multiple ranks, and sometimes signals that are subtle at species-level become clearer at higher ranks.</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.4"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.4.1" style="font-size:90%;">Another core concept is relative abundance. In most metagenomics community profiling settings, especially when comparing conditions, we care about how large a fraction of the community is represented by each taxon relative to the total. Relative abundance is typically expressed as a proportion or percentage: for example, ‘‘Taxon X constitutes 12% of detected bacterial reads in JC1A.’’ Relative abundance is valuable because it supports comparisons even when the total amount of sequence data differs between samples, as it normalizes by the total measured signal. However, relative abundance is also inherently compositional: if one taxon’s relative abundance increases, others must collectively decrease to keep the total at 100%. This means interpretations should focus on changes in community composition rather than assuming that every increase corresponds to absolute growth (unless absolute measurements are provided, which they are not here).</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.5"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.5.1" style="font-size:90%;">When thinking about nutrient enrichment in microbial ecosystems, it is helpful to frame it as a broad ecological perturbation. Nutrients can act as limiting resources; when they become more available, microbes that can quickly exploit them may become more prevalent, potentially outcompeting taxa adapted to nutrient-poor conditions. Conversely, taxa that are specialized for low-nutrient environments may decline in relative representation if the enriched environment favors different metabolic strategies. Nutrient changes can also indirectly influence community structure by shifting interactions such as cross-feeding, competition, and niche partitioning. Importantly, the task does not require any narrative ecological interpretation; it focuses on accurately describing which bacterial taxa are present and their relative abundances in each condition.</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.6"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.6.1" style="font-size:90%;">It is also useful to understand what is meant by ‘‘profiling’’ in this context. Profiling means producing a structured inventory of bacterial taxa detected in each sample condition and quantifying their relative abundances. It does not imply mechanistic interpretation, functional annotation, or inference of metabolic pathways. The objective is not to speculate about why a taxon is present, nor to infer environmental parameters. Instead, it is an exercise in summarizing and comparing community membership and proportional representation in a consistent way.</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.7"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.7.1" style="font-size:90%;">The term ‘‘bacterial taxa’’ indicates that the report should focus specifically on bacteria rather than other biological groups. In mixed microbial datasets, there can be signals from non-bacterial sources; however, the reporting target here is bacterial composition. Conceptually, bacteria are one of the dominant microbial domains in many environmental communities, and bacterial taxonomic summaries are a standard output for metagenomics comparisons. The report should reflect bacterial taxa and their relative abundances, capturing the compositional profile of the community for each condition.</span></span> <span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.8"><span id="A1.SS8.SSS0.Px1.p4.pic1.1.1.1.1.1.8.1" style="font-size:90%;">Because the data come from natural samples, it is worth keeping in mind some general properties of environmental sequencing-derived read collections. Environmental samples can contain uneven distributions of organisms: a small number of taxa may dominate while many others are present at low levels. Additionally, detection and apparent abundance can be influenced by biological and sampling factors such as community heterogeneity and stochastic sampling of rare taxa. Again, for this task, these are background considerations to support careful reporting, not instructions to perform any specific procedure.</span></span></span></foreignObject></g></g></svg>

#### Corrupt inputs.

We synthetically corrupt selected input files and measure whether the agent detects the corruption and avoids proceeding with invalid data.

- alzheimer-mouse: synthetic differential expression table with constant/uninformative values (DEA\_PS3O1S.csv;
- comparative-genomics: inserted 1600 A bases in random places in 4.fna files.
- cystic-fibrosis: scramble the metadata.
- deseq:.fastq files with $\sim 90\%$ bases replaced by N and all quality scores set to "!" (Phred 0).
- evolution: ancestor \*.fastq.gz (only files containing "anc") with $\sim 90\%$ bases replaced by N and all quality scores set to "!" (Phred 0).
- giab: \*.fastq.gz with $\sim 90\%$ bases replaced by N and all quality scores set to "!" (Phred 0).
- metagenomics: \*.fastq.gz with $\sim 90\%$ bases replaced by N and all quality scores set to "!" (Phred 0).
- single-cell:.mtx entries rewritten to constant 777.
- transcript-quant: \*.fastq.gz with $\sim 90\%$ bases replaced by N and all quality scores set to "!" (Phred 0).
- viral-metagenomics — \*.fastq.gz with $\sim 90\%$ bases replaced by N and all quality scores set to "!" (Phred 0).

Decoy inputs. We inject decoy inputs (e.g., sequences from an unrelated organism) to test whether the agent can correctly contextualize the data and exclude irrelevant files from downstream analysis.

- alzheimer-mouse: inserted synthetic mouse differential expression table with randomized gene ids/names and randomized statistics.
- comparative-genomics: inserted genomic sequence from E.Coli
- cystic-fibrosis: inserted random VCF
- deseq: inserted genomic sequence and scaffold from Candida tropicalis strain
- evolution: inserted control library.fastq.gz with N-only reads (length 150) and Phred 0;
- giab: inserted NA12877\_R1/R2.fq.gz 150k read pairs (150 bp) from GRCh38 reference
- metagenomics: inserted a Kraken viral database.
- single-cell: inserted cell marker metadata with obviously wrong values.
- transcript-quant: inserted short random transcriptome FASTA.
- viral-metagenomics: inserted reference genome for Delphinapterus Leucas

[^1]: AnomalyInnovations OpenCode. Note: [https://github.com/anomalyco/opencode](https://github.com/anomalyco/opencode) Cited by: §5.1.

[^2]: Anthropic Claude code. Note: [https://github.com/anthropics/claude-code](https://github.com/anthropics/claude-code) Cited by: §5.1.

[^3]: A program for annotating and predicting the effects of single nucleotide polymorphisms, snpeff: snps in the genome of drosophila melanogaster strain w1118; iso-2; iso-3. Fly 6 (2), pp. 80–92. Cited by: 3rd item.

[^4]: H. Gourlé External Links: [Link](https://www.hadriengourle.com/tutorials/metavir/) Cited by: 10th item.

[^5]: Demystifying evals for ai agents. Note: Anthropic Engineering BlogPublished Jan 09, 2026. Accessed 2026-01-28 External Links: [Link](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) Cited by: §A.8, §3.

[^6]: StableToolBench: towards stable large-scale benchmarking on tool learning of large language models. In Findings of the Association for Computational Linguistics ACL 2024, pp. 11143–11156. Cited by: §2.

[^7]: Comparative phenotypic analysis of the major fungal pathogens Candida parapsilosis and Candida albicans. PLoS Pathogens 10 (9), pp. e1004365. External Links: [Document](https://dx.doi.org/10.1371/journal.ppat.1004365) Cited by: 4th item.

[^8]: SWE-bench: can language models resolve real-world github issues?. In The Twelfth International Conference on Learning Representations, External Links: [Link](https://openreview.net/forum?id=VTF8yNQM66) Cited by: §2, §5.1.

[^9]: CompGenomicsBioc2022: comparative genomics analyses with synextend and decipher. Note: R package version 3.4.0 External Links: [Link](https://www.ahl27.com/CompGenomicsBioc2022/) Cited by: 2nd item.

[^10]: LAB-bench: measuring capabilities of language models for biology research. External Links: 2407.10362, [Link](https://arxiv.org/abs/2407.10362) Cited by: §2.

[^11]: API-bank: a comprehensive benchmark for tool-augmented llms. In EMNLP, Cited by: §2.

[^12]: AgentBench: evaluating llms as agents. In ICLR, Cited by: §2.

[^13]: Benchmarking llm-based agents for single-cell omics analysis. External Links: 2508.13201, [Link](https://arxiv.org/abs/2508.13201) Cited by: §2.

[^14]: Single-cell sequencing deconvolutes cellular responses to exercise in human skeletal muscle. Communications Biology 5, pp. 1121. External Links: [Document](https://dx.doi.org/10.1038/s42003-022-04088-z), [Link](https://doi.org/10.1038/s42003-022-04088-z) Cited by: 8th item.

[^15]: BioML-bench: evaluation of ai agents for end-to-end biomedical ml. bioRxiv. External Links: [Document](https://dx.doi.org/10.1101/2025.09.01.673319) Cited by: §2.

[^16]: BixBench: a comprehensive benchmark for llm-based agents in computational biology. External Links: 2503.00096, [Link](https://arxiv.org/abs/2503.00096) Cited by: §2.

[^17]: OpenAI Codex CLI. Note: [https://github.com/openai/codex](https://github.com/openai/codex) Cited by: §5.1.

[^18]: E. Peikon Mouse\_Alz\_Models. Note: GitHub repository External Links: [Link](https://github.com/evanpeikon/Mouse_Alz_Models) Cited by: 1st item.

[^19]: A universal SNP and small-indel variant caller using deep neural networks. Nature Biotechnology 36, pp. 983–987. External Links: [Document](https://dx.doi.org/10.1038/nbt.4235) Cited by: §5.2.

[^20]: Scaling accurate genetic variant discovery to tens of thousands of samples. bioRxiv. External Links: [Document](https://dx.doi.org/10.1101/201178), [Link](https://www.biorxiv.org/content/early/2018/07/24/201178), https://www.biorxiv.org/content/early/2018/07/24/201178.full.pdf Cited by: §5.2.

[^21]: Tool learning with foundation models. ACM Computing Surveys 57 (4), pp. 1–40. Cited by: §2.

[^22]: ToolLLM: facilitating large language models to master 16000+ real-world apis. In ICLR, Cited by: §2.

[^23]: Devstral: fine-tuning language models for coding agent applications. External Links: 2509.25193, [Link](https://arxiv.org/abs/2509.25193) Cited by: §A.8.

[^24]: Computational genomics tutorial. External Links: [Link](https://genomics.sschmeier.com/) Cited by: 5th item.

[^25]: GLM-4.5: agentic, reasoning, and coding (arc) foundation models. External Links: 2508.06471, [Link](https://arxiv.org/abs/2508.06471) Cited by: §A.8.

[^26]: Kimi k2: open agentic intelligence. External Links: 2507.20534, [Link](https://arxiv.org/abs/2507.20534) Cited by: §A.8.

[^27]: Reproducible, scalable, and shareable analysis pipelines with bioinformatics workflow managers. Nature Methods 18, pp. 1161–1168. External Links: [Document](https://dx.doi.org/10.1038/s41592-021-01254-9), [Link](https://doi.org/10.1038/s41592-021-01254-9) Cited by: 9th item.

[^28]: Patient privacy in AI-driven omics methods. Trends in Genetics 40 (5), pp. 383–386. External Links: [Document](https://dx.doi.org/10.1016/j.tig.2024.03.004) Cited by: §1.

[^29]: A data carpentry- style metagenomics workshop. Journal of Open Source Education 7 (72), pp. 209. External Links: [Document](https://dx.doi.org/10.21105/jose.00209), [Link](https://doi.org/10.21105/jose.00209) Cited by: 7th item.

[^30]: Integrating human sequence data sets provides a resource of benchmark SNP and indel genotype calls. Nature Biotechnology 32 (3), pp. 246–251. External Links: [Document](https://dx.doi.org/10.1038/nbt.2835) Cited by: 6th item.