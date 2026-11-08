---
name: ionomics-data-processing
description: Use when raw ICP-MS ion-concentration data (ppm values, optionally OD-adjusted,
  with batch structure and replicate/control mutants such as YDL227C) must be cleaned,
  batch-corrected and standardised before multivariate ionome analysis.
when_to_use_negative:
- The data are already normalised and standardised — IonFlow 'can be used to process
  raw data as well as directly process normalised and standardised data', so pre-processing
  would double-correct
- The input is not ICP-MS element-concentration data (e.g. transcriptomics or spectral
  data) outside the ionomics scope of the pipeline
- Only downstream analysis is needed and a cleaned dataset already exists — skip to
  the exploratory, clustering or network sections directly
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IonFlow
  role: R package/Galaxy tool implementing the pre-processing section (outlier detection,
    median batch correction, standardisation) plus PCA, clustering and network analysis
  repo: https://github.com/wanchanglin/ionflow
- name: MetaboFlow (IonFlow R package)
  role: Underlying R package whose PreProcessing function performs the cleaning steps;
    installable via devtools::install_github
  repo: https://github.com/AlinaPeluso/MetaboFlow
- name: Galaxy
  role: Platform wrapping IonFlow as a user-friendly tool; installed via the Galaxy
    Tool Shed and registered in tool_conf.xml
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:3c6cbb76a9ba191bdf5bc09f9e56457db2c41f5992fb6b66242822209e41ced9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - MetaboFlow (IonFlow R package)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ionomics-data-processing

## Summary

Pre-process raw ICP-MS ionomics concentration data (e.g. OD-corrected yeast deletion-mutant ionomes) using IonFlow's pipeline: outlier removal via 3-IQR outer fences, log-transform, per-batch median normalisation, standardisation, and ±2 z-score symbolisation, to yield a cleaned dataset for PCA, clustering, relevance networks and enrichment.

## When to use

Use when raw ICP-MS ion-concentration data (ppm values, optionally OD-adjusted, with batch structure and replicate/control mutants such as YDL227C) must be cleaned, batch-corrected and standardised before multivariate ionome analysis.

## When NOT to use

- The data are already normalised and standardised — IonFlow 'can be used to process raw data as well as directly process normalised and standardised data', so pre-processing would double-correct
- The input is not ICP-MS element-concentration data (e.g. transcriptomics or spectral data) outside the ionomics scope of the pipeline
- Only downstream analysis is needed and a cleaned dataset already exists — skip to the exploratory, clustering or network sections directly

## Inputs

- Raw ICP-MS ion-concentration data frame (ppm, OD-corrected; e.g. 14 ions across yeast haploid knockouts with 2/4/8 replicates per mutant)
- Optional pre-defined per-ion standard deviations computed from control strains/genes
- Batch/plate structure metadata (control mutant YDL227C measured in every batch)

## Outputs

- Cleaned, batch-corrected and standardised ionome dataset (pre_proc$stats.median_batch_corrected_data-style table)
- Per-ion raw-data statistics (min, quartiles, median, mean, max, variance)
- Per-ion outlier counts and outlier percentages
- Symbolised (±2 z-score) ionome profiles feeding PCA, hierarchical clustering, relevance networks and gene set enrichment

## How to apply

Load the raw ion-concentration data frame (e.g. IonData with 14 ions: Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66) and run PreProcessing, optionally supplying pre-defined ion standard deviations computed from control genes; if omitted, SDs are derived from the data. Remove extreme outliers using Tukey-style outer fences at Q1 - 3*IQ and Q3 + 3*IQ per ion (documented outlier rates range from 0.01% for Mg/P/K to 0.22% for Ca). Then log-transform concentrations and scale to the per-ion, per-batch median (median batch correction) to remove batch effects, exploiting a control strain measured in every batch (YDL227C). Standardise the corrected values (z-score symbolisation at a ±2 threshold per the iHUB case study) and pass the cleaned dataset to downstream IonFlow sections: PCA, hierarchical clustering with GO Slim annotation/GO enrichment, and relevance-network/correlation analysis.

## Related tools

- **IonFlow** (R package/Galaxy tool implementing the pre-processing section (outlier detection, median batch correction, standardisation) plus PCA, clustering and network analysis) — https://github.com/wanchanglin/ionflow
- **MetaboFlow (IonFlow R package)** (Underlying R package whose PreProcessing function performs the cleaning steps; installable via devtools::install_github) — https://github.com/AlinaPeluso/MetaboFlow
- **Galaxy** (Platform wrapping IonFlow as a user-friendly tool; installed via the Galaxy Tool Shed and registered in tool_conf.xml) — https://github.com/galaxyproject/galaxy

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- Post-correction per-ion median equals exactly 0 within each batch (median_batch_corrected_data stats table shows Median = 0 for all 14 ions)
- Per-ion outlier fractions fall in the documented 0.01%–0.22% range under the 3-IQR outer-fence rule (e.g. Mg: 8/9999; Ca: 305/9999)
- Raw-data statistics table (Min, 1st Quartile, Median, Mean, 3rd Quartile, Max, Variance per ion) reproduces expected ranges (e.g. P median ≈ 4514 ppm, Mg median ≈ 679 ppm)
- Element-element correlation analysis on cleaned data recovers the expected high Mg–P correlation reported for the iHUB yeast case study
- Symbolised-profile invariance check: ~75% (3171/4945) of haploid deletion mutants show no change in symbolic ionome profile at the selected threshold

## Limitations

- Outlier removal at Q1 - 3*IQ / Q3 + 3*IQ is per-ion and can flag up to ~22% of values for highly variable ions (e.g. Ca), so aggressive trimming may bias rare extreme phenotypes
- No version numbers stated for IonFlow or Galaxy; no quantitative benchmark performance metrics reported in the paper's discussion
- If pre-defined standard deviations from control genes are not supplied, data-derived SDs are used instead, which may change downstream symbolisation and unchanged-profile fractions

## Evidence

- [abstract] IonFlow is a Galaxy tool written in R for ionomics data analysis: "IonFlow is a Galaxy tool written in R for ionomics data analysis"
- [intro] developed IonFlow for ICP-MS ionome data: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
- [discussion] IonFlow handles raw and pre-normalised data: "IonFlow is very versatile and it can be used to process raw data as well as directly process normalised and standardised data"
- [readme] outer-fence outlier definition: [evidence span withheld — anchor status: external_doc]
- [readme] median batch correction step: [evidence span withheld — anchor status: external_doc]
- [readme] pre-processing purpose statement: [evidence span withheld — anchor status: external_doc]
