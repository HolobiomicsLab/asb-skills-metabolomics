---
name: statistical-data-verification
description: Use when a task asks to reproduce or verify published statistical claims
  from the IonFlow yeast ionomics case study (unchanged-profile fraction, ion correlations,
  network centrality extremes, enrichment results) against a re-executed pipeline.
when_to_use_negative:
- The input is a metabolomics dataset or a non-yeast organism's ionome — the verified
  statistics (4945 mutants, 75% unchanged, Mg–P correlation, SMF2/MTF1) are specific
  to the haploid S. cerevisiae ionome benchmark
- The data are already normalised/standardised beyond what IonFlow's pre-processing
  expects — the tool can process raw data directly, but feeding pre-processed data
  through the plate-median/±2 z-score pipeline would break comparability with the
  published thresholds
- The goal is a new biological discovery rather than verification of the published
  case-study numbers — use the exploratory IonFlow sections directly instead of the
  verification checklist
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: IonFlow
  role: R-based pipeline executed end-to-end (pre-processing, PCA, hierarchical clustering,
    relevance network extraction, GO enrichment) to regenerate the yeast case-study
    statistics
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Execution environment in which the IonFlow tool is installed via tool_conf.xml
    or the Galaxy Tool Shed to run the case-study workflow
  repo: https://github.com/galaxyproject/galaxy
- name: ICP-MS
  role: Inductively coupled plasma mass spectrometry — the measurement technology
    that produced the ion concentration profiles being verified
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:9c5e95b99b818490332d6048901982beb328979896ed28c2728bcdedfa735308
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - ICP-MS
  - IonFlow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# statistical-data-verification

## Summary

Re-run the IonFlow pipeline on the iHUB yeast ionome dataset and verify the published case-study statistics — the ~75% unchanged-profile fraction among 4945 haploid deletion mutants, exponential decay in the number of altered ions, Mg–P correlation, SMF2/MTF1 network extremes, and mitochondrial gene-set enrichment.

## When to use

Use when a task asks to reproduce or verify published statistical claims from the IonFlow yeast ionomics case study (unchanged-profile fraction, ion correlations, network centrality extremes, enrichment results) against a re-executed pipeline.

## When NOT to use

- The input is a metabolomics dataset or a non-yeast organism's ionome — the verified statistics (4945 mutants, 75% unchanged, Mg–P correlation, SMF2/MTF1) are specific to the haploid S. cerevisiae ionome benchmark
- The data are already normalised/standardised beyond what IonFlow's pre-processing expects — the tool can process raw data directly, but feeding pre-processed data through the plate-median/±2 z-score pipeline would break comparability with the published thresholds
- The goal is a new biological discovery rather than verification of the published case-study numbers — use the exploratory IonFlow sections directly instead of the verification checklist

## Inputs

- iHUB Yeast Ionome OD-corrected ICP-MS dataset (4945 haploid deletion mutants, 14 ions: Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn; concentrations in ppm, optical-density adjusted)
- Optionally pre-defined ion standard deviations (e.g. computed from the control strain YDL227C measured in every batch)
- IonFlow Galaxy tool configuration (tool_conf.xml entry) or the IonFlow R package

## Outputs

- Symbolic ionome profile per mutant with unchanged/changed classification and the unchanged-profile fraction (expected ~3171/4945, ~75%)
- Distribution of the number (k) of altered ions per phenotypic mutant line (expected exponential decay)
- Element-element correlation matrix / relevance network, including the Mg–P correlation
- Gene impact scores (SMF2) and network betweenness centralities (MTF1)
- Gene set enrichment results with significantly enriched biological processes (mitochondrial terms)

## How to apply

Load the iHUB Yeast Ionome OD-corrected ICP-MS dataset (14 ions: Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn) into the IonFlow Galaxy tool or the underlying R package. Run IonFlow's pre-processing chain — plate-median normalisation, 3-std outlier removal, and ±2 z-score symbolisation — then execute its four analysis sections: PCA, hierarchical clustering, relevance network extraction with element-element correlation analysis, and gene set (GO) enrichment. Compare each re-computed statistic against the published benchmarks: (1) 3171 of 4945 (~75%) haploid deletion mutants show no change in their symbolic ionome profile at the selected threshold; (2) for the phenotypic lines, the probability of k altered ions decays exponentially with k; (3) Mg and P show a high correlation in the relevance network; (4) SMF2 receives extreme impact scores and MTF1 has extreme network betweenness; (5) significant clusters map to mitochondrial biological processes. Treat any material deviation in these quantities as a signal that pre-processing parameters or thresholds differ from the published configuration.

## Related tools

- **IonFlow** (R-based pipeline executed end-to-end (pre-processing, PCA, hierarchical clustering, relevance network extraction, GO enrichment) to regenerate the yeast case-study statistics) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Execution environment in which the IonFlow tool is installed via tool_conf.xml or the Galaxy Tool Shed to run the case-study workflow) — https://github.com/galaxyproject/galaxy
- **ICP-MS** (Inductively coupled plasma mass spectrometry — the measurement technology that produced the ion concentration profiles being verified)

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- Unchanged-profile fraction: 3171 of 4945 haploid deletion mutants (~75%) must show no change in their symbolic ionome profile at the selected threshold
- Phenotypic lines only: probability of having k altered ions decreases exponentially with k
- Mg–P correlation in the relevance network must be high, as reported in the case study
- SMF2 must appear among extreme impact scores and MTF1 must show extreme network betweenness
- Gene set enrichment must recover mitochondrial biological processes among the statistically significant clusters

## Limitations

- The article reports no quantitative benchmark results or performance metrics for the two case studies in the main text; the verified statistics come from the case-study findings, so re-implementation details (threshold values, symbolisation cutoffs) must match exactly to reproduce 3171/4945
- The benchmark yeast dataset described in the README comprises 1454 single-gene haploid knockouts (Danku 2009), which differs in scale from the 4945 iHUB mutants in the research question — confirm which dataset version is loaded before comparing counts
- No explicit limitations or caveats are discussed in the article's discussion section, and no version numbers for IonFlow or Galaxy are stated, complicating exact reproducibility

## Evidence

- [abstract] The pipeline is described and tested on two benchmark data sets of the haploid S. Cerevisiae ionome and of the human HeLa cell ionome.: "The pipeline is described and tested on two benchmark data sets of the haploid S. Cerevisiae ionome and of the human HeLa cell ionome."
- [abstract] PCA, hierarchical clustering, relevance networks and gene set enrichment are the verification targets.: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis, and gene set enrichment analysis"
- [readme] README documents the yeast ICP-MS data used by the workflow, grounding the verification inputs.: [evidence span withheld — anchor status: external_doc]
- [readme] README notes batch and ion variability, motivating the pre-processing steps whose parameters must match for verification.: [evidence span withheld — anchor status: external_doc]
- [discussion] Article discussion lists the multivariate analyses whose outputs are checked against published findings.: "multivariate approaches used in the field of ionomics including PCA, correlation analysis, network inference and enrichment analysis"
- [readme] README shows the concrete R invocation for the pre-processing step that must be reproduced.: [evidence span withheld — anchor status: external_doc]
