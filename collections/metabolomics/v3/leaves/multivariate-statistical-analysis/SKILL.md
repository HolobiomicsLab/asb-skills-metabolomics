---
name: multivariate-statistical-analysis
description: 'Use when a pre-processed (batch-corrected, log-transformed, standardised)
  ICP-MS ionome table of ion concentrations needs exploratory multivariate analysis:
  variance decomposition, element-element correlation networks, or clustered correlation
  heat maps.'
when_to_use_negative:
- Data have not passed through the PreProcessing section — the pipeline requires the
  cleaned dataset from PreProcessing before exploratory analysis, so raw, outlier-laden
  concentration tables should not be fed directly to these analysis steps
- The goal is metabolomics feature or spectral analysis rather than ion-concentration
  (ionomics) profiles — although the authors note applicability may extend to metabolomics,
  the tool was designed and benchmarked on ICP-MS ionome data
- Only a single-variable summary or univariate group comparison is needed — the suite
  is built for multivariate variance structure, correlation, and network inference
  across many ions
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: IonFlow
  role: R package executed for PCA, hierarchical clustering, relevance network extraction
    and correlation heat-mapping on the ionome dataset
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: User-friendly interface through which the IonFlow multivariate analysis workflow
    is exposed to users
  repo: https://github.com/galaxyproject/galaxy
- name: R
  role: Implementation language of the IonFlow package that carries out the multivariate
    statistical analyses
  repo: https://github.com/AlinaPeluso/MetaboFlow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:0f03210db899693556d37952a5cfe2093ca86aea4fb19453ea3fd4985b44ec85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multivariate-statistical-analysis

## Summary

Applies the multivariate statistics suite of the IonFlow R package — PCA, element-element correlation analysis, Ward-based hierarchical clustering, relevance network extraction, and correlation heat-mapping — to ICP-MS ionome profiles from ionomics experiments.

## When to use

Use when a pre-processed (batch-corrected, log-transformed, standardised) ICP-MS ionome table of ion concentrations needs exploratory multivariate analysis: variance decomposition, element-element correlation networks, or clustered correlation heat maps.

## When NOT to use

- Data have not passed through the PreProcessing section — the pipeline requires the cleaned dataset from PreProcessing before exploratory analysis, so raw, outlier-laden concentration tables should not be fed directly to these analysis steps
- The goal is metabolomics feature or spectral analysis rather than ion-concentration (ionomics) profiles — although the authors note applicability may extend to metabolomics, the tool was designed and benchmarked on ICP-MS ionome data
- Only a single-variable summary or univariate group comparison is needed — the suite is built for multivariate variance structure, correlation, and network inference across many ions

## Inputs

- Pre-processed ionome dataset (cleaned, log-transformed, batch-corrected, standardised ion concentrations from ICP-MS), e.g. data(IonData) in the IonFlow R package
- Correlation threshold parameter (thres_ion_corr)
- Correlation method parameter (method_ion_corr)
- Optional pre-defined ion standard deviations (otherwise standard deviations are computed from the data)

## Outputs

- PCA results showing the most significant variance components of the ionome data
- Element-element correlation matrix
- Relevance network (plot.net output) with element nodes and link width proportional to the correlation coefficient
- Hierarchical clustering of elements (Ward's method, Euclidean distance)
- Correlation heat map ordered by the resulting hierarchical clustering

## How to apply

Load the line-aggregated z-score profiles and specify the correlation thresholds thres_ion_corr and method_ion_corr within IonFlow. First run principal components analysis (PCA) to identify the most significant variance in the ionome data across the measured ions (e.g. the 14 ions Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66 in the S. cerevisiae benchmark). Next compute element-element correlations on the input profiles using the chosen method and threshold, then perform hierarchical clustering with Ward's method (ward.D) and Euclidean distance to group related elements. Build the relevance network by linking only element pairs whose correlation exceeds the threshold, and render it via plot.net where node = element and link width is proportional to the correlation coefficient. Finally generate the correlation heat map from the hierarchical clustering and the correlation matrix; note the PreProcessing section must be run first to produce the cleaned dataset used by these analysis sections.

## Related tools

- **IonFlow** (R package executed for PCA, hierarchical clustering, relevance network extraction and correlation heat-mapping on the ionome dataset) — https://github.com/wanchanglin/ionflow
- **Galaxy** (User-friendly interface through which the IonFlow multivariate analysis workflow is exposed to users) — https://github.com/galaxyproject/galaxy
- **R** (Implementation language of the IonFlow package that carries out the multivariate statistical analyses) — https://github.com/AlinaPeluso/MetaboFlow

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow)
```

## Evaluation signals

- The relevance network contains edges only between element pairs whose correlation coefficient exceeds the specified thres_ion_corr threshold
- In the plot.net output, link width between two elements is proportional to their correlation coefficient, and nodes correspond one-to-one with the measured elements
- The correlation heat map's dendrogram is consistent with the hierarchical clustering computed with Ward's method and Euclidean distance
- PCA components are ordered so the leading components capture the most significant variance in the ionome data
- The correlation matrix is symmetric with unit diagonal and derived from the standardised (z-score) input profiles

## Limitations

- No quantitative benchmark results or performance metrics were reported for the two case studies (S. cerevisiae and HeLa ionomes)
- No explicit limitations or caveats are discussed in the paper's discussion section
- The PreProcessing section must be run first to produce the cleaned dataset; the analysis sections can follow in any order after it
- The paper notes ion-profile-specific similarity measures (Iacovacci et al., 2020) are only optionally available to the user

## Evidence

- [abstract] Pipeline covers PCA, clustering, networks: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis"
- [abstract] Benchmark datasets for the pipeline: "The pipeline is described and tested on two benchmark data sets of the haploid S. Cerevisiae ionome and of the human HeLa cell ionome."
- [readme] IonFlow is an R package with four workflow sections: [evidence span withheld — anchor status: external_doc]
- [readme] Ions measured in the yeast benchmark dataset: [evidence span withheld — anchor status: external_doc]
- [discussion] Multivariate approaches used in ionomics: "multivariate approaches used in the field of ionomics including PCA, correlation analysis, network inference and enrichment analysis"
- [intro] Purpose of the IonFlow tool: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
