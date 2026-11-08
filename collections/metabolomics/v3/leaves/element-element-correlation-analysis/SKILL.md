---
name: element-element-correlation-analysis
description: Use when a pre-processed ICP-MS ionome matrix (e.g. S. cerevisiae mutants
  profiled across ions like Ca44, K39, Mg25, P31, Zn66 in ppm) must be examined for
  element-element correlations and relevance networks.
when_to_use_negative:
- Input is raw ICP-MS spectral output rather than tabulated ion concentration values
  in ppm — IonFlow operates on concentration data frames, not instrument spectra
- The dataset has no batch structure or replicate measurements, so median batch correction
  and control-strain-based standardisation cannot be meaningfully applied
- The goal is element quantification or assay curation rather than studying inter-element
  relationships and network structure
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: IonFlow
  role: R package/Galaxy tool performing correlation analysis, relevance network extraction
    and enrichment on the cleaned ionome matrix
  repo: github.com/wanchanglin/ionflow
- name: Galaxy
  role: Execution environment exposing the IonFlow tool via a user-friendly web interface
    (tool_conf.xml registration)
  repo: https://github.com/galaxyproject/galaxy
- name: R
  role: Language in which the IonFlow package implementing correlation and network
    analysis is written
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:b096f2d4d18ddc21acc69ec9b2ab9882521618b3439616cd2b60474b51f09270
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# element-element-correlation-analysis

## Summary

Compute element-element correlations and extract relevance networks from ICP-MS ionome concentration data using IonFlow's R-based Galaxy pipeline, to quantify physiological relationships between elements (e.g. the high Mg-P correlation) across mutant profiles.

## When to use

Use when a pre-processed ICP-MS ionome matrix (e.g. S. cerevisiae mutants profiled across ions like Ca44, K39, Mg25, P31, Zn66 in ppm) must be examined for element-element correlations and relevance networks.

## When NOT to use

- Input is raw ICP-MS spectral output rather than tabulated ion concentration values in ppm — IonFlow operates on concentration data frames, not instrument spectra
- The dataset has no batch structure or replicate measurements, so median batch correction and control-strain-based standardisation cannot be meaningfully applied
- The goal is element quantification or assay curation rather than studying inter-element relationships and network structure

## Inputs

- OD-corrected ICP-MS ion concentration matrix (ppm) for haploid S. cerevisiae knockouts across 14 ions (Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66)
- Optional pre-defined ion standard deviations computed from control strains (e.g. repeated measurements of YDL227C)
- Batch/replicate structure metadata (2, 4, or 8 replicas per mutant, one control strain measured in every batch)

## Outputs

- Element-element correlation matrix over the 14 ion profiles
- Extracted relevance network with node-level statistics (e.g. SMF2 impact scores, MTF1 betweenness)
- Median batch-corrected and outlier-filtered cleaned dataset
- Gene set / GO term enrichment results for significant clusters

## How to apply

Run IonFlow's PreProcessing first: log-transform ppm concentrations, remove extreme outliers defined by outer fences Q1 - 3*IQ and Q3 + 3*IQ, and scale each ion to its median within each batch (optionally using a pre-defined standard deviation from the control strain YDL227C instead of data-derived SDs). On the cleaned dataset, perform correlation analysis across the 14 element profiles to evaluate elemental relationships, explicitly checking for high correlation between Mg and P as a validation landmark. Feed significant correlations into relevance network extraction and gene set enrichment analysis to identify statistically significant clusters and associated biological processes. Validate network results against known biological anchors such as SMF2 impact scores and MTF1 network betweenness. Note that IonFlow can also accept directly normalised/standardised data, so pre-processing can be skipped if such input is supplied.

## Related tools

- **IonFlow** (R package/Galaxy tool performing correlation analysis, relevance network extraction and enrichment on the cleaned ionome matrix) — github.com/wanchanglin/ionflow
- **Galaxy** (Execution environment exposing the IonFlow tool via a user-friendly web interface (tool_conf.xml registration)) — https://github.com/galaxyproject/galaxy
- **R** (Language in which the IonFlow package implementing correlation and network analysis is written)

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- After median batch correction, the per-ion median of the corrected data equals 0 (as shown in the pre_proc$stats.median_batch_corrected_data table)
- Outlier counts per ion match the outer-fence removal table (e.g. Ca: 305/9999 outliers = 0.22%; Mg and P: 8 outliers = 0.01%)
- A high Mg-P correlation is recovered in the element-element correlation analysis, matching the case-study expectation
- Network statistics reproduce the reference values: SMF2 impact scores and MTF1 network betweenness extremes from the yeast iHUB case study
- The dataset contains no missing values, so any NA introduction signals a pre-processing error

## Limitations

- No quantitative benchmark results or performance metrics are reported for the two case studies in the discussion
- No version numbers are stated for IonFlow or Galaxy, hindering exact reproducibility
- Supplementary material is referenced only by file size (PDF 1754 kb) without a description of contents, so some method parameters may be undocumented

## Evidence

- [abstract] Pipeline performs correlation and relevance network analysis among ionomics multivariate methods: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis"
- [discussion] Correlation analysis is listed among the ionomics multivariate approaches supported by IonFlow: "multivariate approaches used in the field of ionomics including PCA, correlation analysis, network inference and enrichment analysis"
- [intro] IonFlow was designed specifically for ICP-MS ionome datasets, the input type this skill operates on: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
- [readme] README describes the four-section IonFlow workflow structure on which the correlation step depends: [evidence span withheld — anchor status: external_doc]
- [readme] README grounds the example input data (yeast ICP-MS, 14 ions, ppm, OD-corrected) used in correlation analysis: [evidence span withheld — anchor status: external_doc]
- [readme] README defines the outlier-removal threshold (3*IQ outer fences) applied before correlation analysis: [evidence span withheld — anchor status: external_doc]
