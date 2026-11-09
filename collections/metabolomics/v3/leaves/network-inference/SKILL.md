---
name: network-inference
description: Use when ICP-MS ion-concentration data (e.g. haploid S. cerevisiae deletion-mutant
  ionomes) have been normalised/symbolised and you need element-element or mutant-mutant
  network extraction plus correlation analysis.
when_to_use_negative:
- The input is raw, uncorrected ICP-MS output that has not passed through IonFlow's
  pre-processing (outlier removal and median batch correction) — network edges would
  reflect batch effects
- The analysis goal is purely exploratory dimensionality reduction with no relational/network
  question (PCA or hierarchical clustering alone suffices)
- The data are not elemental-concentration profiles (IonFlow's applicability to metabolomics
  is stated as only potential, not demonstrated)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: IonFlow
  role: R package providing relevance network extraction, ion-profile-specific similarity
    measures, and element-element correlation analysis on ionome data
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Platform through which the IonFlow network-analysis tools are exposed with
    a user-friendly interface
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:c2c11b90a7734cd9b5e6409ea94c18fa27fa6abce3d6691b05511839737a4bc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# network-inference

## Summary

Extract and analyse relevance networks from ionome profiles measured by ICP-MS, using ion-profile-specific similarity measures to link mutants or elements, as implemented in the IonFlow R/Galaxy pipeline.

## When to use

Use when ICP-MS ion-concentration data (e.g. haploid S. cerevisiae deletion-mutant ionomes) have been normalised/symbolised and you need element-element or mutant-mutant network extraction plus correlation analysis.

## When NOT to use

- The input is raw, uncorrected ICP-MS output that has not passed through IonFlow's pre-processing (outlier removal and median batch correction) — network edges would reflect batch effects
- The analysis goal is purely exploratory dimensionality reduction with no relational/network question (PCA or hierarchical clustering alone suffices)
- The data are not elemental-concentration profiles (IonFlow's applicability to metabolomics is stated as only potential, not demonstrated)

## Inputs

- Cleaned ICP-MS ion-concentration matrix (e.g. OD-corrected S. cerevisiae ionome data with 14 ions: Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66)
- Pre-processed dataset output by the IonFlow pre-processing section (log-transformed, median-batch-corrected, standardised, optionally symbolised)
- Optional pre-defined ion standard deviations computed from control strains (e.g. YDL227C)

## Outputs

- Relevance network linking mutants/elements with similarity-based edges
- Element-element correlation matrix (including the Mg–P correlation result)
- Network node statistics such as betweenness centrality (e.g. MTF1) and impact scores (e.g. SMF2)
- Enrichment-annotated clusters of phenotypic lines (GO Slim / GO terms)

## How to apply

Run network analysis in the IonFlow R package (Galaxy-wrapped) after pre-processing has produced the cleaned, median-batch-corrected dataset. Build the relevance network using the ion-profile-specific similarity measures of Iacovacci et al. (2020) — optionally selectable by the user — then perform correlation analysis between elements, checking for high element-element correlations such as the strong Mg–P relationship highlighted in the yeast iHUB case study. Compute network-level statistics on resulting graphs (e.g. betweenness centrality, where MTF1 emerges as an extreme node, and impact scores such as SMF2's). Finally, couple the inferred network with enrichment analysis (GO Slim annotation and GO terms enrichment) to assign biological processes to statistically significant clusters. Networks can be computed on raw data or on already normalised/standardised data, since IonFlow accepts both.

## Related tools

- **IonFlow** (R package providing relevance network extraction, ion-profile-specific similarity measures, and element-element correlation analysis on ionome data) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Platform through which the IonFlow network-analysis tools are exposed with a user-friendly interface) — https://github.com/galaxyproject/galaxy

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- The inferred element-element correlation network recovers the reported high correlation between Mg and P in the yeast data
- Network betweenness/impact statistics reproduce the paper's extremes: MTF1 high betweenness and SMF2 high impact score
- For non-neutral phenotypic lines, the probability of having k altered ions decreases exponentially with k
- Network results are invariant to whether raw versus pre-normalised/standardised input data were supplied, as the tool claims versatility across both
- Edge construction is stable when the ion-profile-specific similarity measure (Iacovacci et al., 2020) is toggled on/off by the user

## Limitations

- No quantitative benchmark results or performance metrics are reported in the paper's discussion for the two case studies
- No explicit limitations or caveats are discussed in the article's discussion section
- No version number is stated for IonFlow or Galaxy, complicating exact reproducibility

## Evidence

- [abstract] Multivariate/network workflow scope: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis"
- [discussion] Similarity measure choice: "ion-profile-specific similarity measures (Iacovacci et al., 2020) that optionally, can be used by the user"
- [discussion] Raw vs normalised input flexibility: "IonFlow is very versatile and it can be used to process raw data as well as directly process normalised and standardised data"
- [readme] R package implementation: [evidence span withheld — anchor status: external_doc]
- [readme] Enrichment coupling: [evidence span withheld — anchor status: external_doc]
