---
name: principal-components-analysis
description: Use when a line-aggregated, z-score standardised ionome dataset (ion
  concentrations from ICP-MS experiments) needs exploratory multivariate analysis
  to reveal the dominant sources of variation across elements/samples.
when_to_use_negative:
- Input is raw, un-batch-corrected concentration data — the PreProcessing section
  (log transform, median batch correction, outlier removal via outer fences Q1-3*IQ
  / Q3+3*IQ) must run first to produce the cleaned dataset.
- The goal is gene set enrichment or GO term analysis rather than exploratory variance
  decomposition — use the clustering/enrichment section of IonFlow instead.
- Data are already normalised metabolomics feature tables outside the ionomics/ICP-MS
  scope — IonFlow's applicability to metabolomics is only 'potential' per the authors.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: IonFlow
  role: R package / Galaxy tool whose IonAnalysis module executes PCA on standardised
    ionome profiles
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web platform providing the user-friendly interface through which the IonFlow
    PCA step is invoked
  repo: https://github.com/galaxyproject/galaxy
- name: R
  role: Implementation language of the IonFlow package carrying out PCA
  repo: https://github.com/wanchanglin/ionflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:2371eca05e8f27a66cb3c884cea4e0a1ef83c19ad261f3bf735f07242dcaa666
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# principal-components-analysis

## Summary

Performs PCA on pre-processed ionome concentration data (e.g. ICP-MS measurements of S. cerevisiae knockout mutants or HeLa cells) to identify the axes of greatest variance across measured ions, as implemented in the IonAnalysis module of the IonFlow Galaxy tool.

## When to use

Use when a line-aggregated, z-score standardised ionome dataset (ion concentrations from ICP-MS experiments) needs exploratory multivariate analysis to reveal the dominant sources of variation across elements/samples.

## When NOT to use

- Input is raw, un-batch-corrected concentration data — the PreProcessing section (log transform, median batch correction, outlier removal via outer fences Q1-3*IQ / Q3+3*IQ) must run first to produce the cleaned dataset.
- The goal is gene set enrichment or GO term analysis rather than exploratory variance decomposition — use the clustering/enrichment section of IonFlow instead.
- Data are already normalised metabolomics feature tables outside the ionomics/ICP-MS scope — IonFlow's applicability to metabolomics is only 'potential' per the authors.

## Inputs

- Line-aggregated z-score standardised ionome profiles (ion concentrations in ppm from ICP-MS, e.g. 14 ions measured across 1454 single-gene haploid S. cerevisiae knockouts)
- Optional pre-defined ion standard deviations computed from control strains (e.g. knock-out YDL227C)

## Outputs

- Principal component scores and loadings summarising the most significant variance in the ionome data
- Exploratory plots of principal components for sample/mutant inspection

## How to apply

Within IonFlow's IonAnalysis module, first load the line-aggregated z-score profiles produced by the PreProcessing section (raw concentrations are log-transformed and median batch-corrected before standardisation). Then run principal components analysis (PCA) to determine the most significant variance in the ionome data, examining how the measured ions (e.g. Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66) contribute to the leading principal components. PCA is one of four IonAnalysis stages and precedes element-element correlation, Ward hierarchical clustering, relevance network construction, and correlation heat-map generation. The rationale is that PCA compresses the high-dimensional ion profile space so that dominant variance patterns (e.g. batch effects or element co-regulation) can be inspected before threshold-dependent network inference is applied.

## Related tools

- **IonFlow** (R package / Galaxy tool whose IonAnalysis module executes PCA on standardised ionome profiles) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Web platform providing the user-friendly interface through which the IonFlow PCA step is invoked) — https://github.com/galaxyproject/galaxy
- **R** (Implementation language of the IonFlow package carrying out PCA) — https://github.com/wanchanglin/ionflow

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- PCA is run on the median batch-corrected, z-scored profiles, not on raw concentrations — verify the input table has median 0 per ion after correction.
- All 14 measured ions (Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66) should appear in the loadings matrix.
- Leading principal components should explain the largest share of variance; check the reported proportion of variance decreases monotonically across components.
- Downstream outputs (element-element correlation network via plot.net, Ward/ward.D heat map) should be consistent with the PCA-reduced structure of the same input matrix.
- Outlier counts per ion should match the PreProcessing stats (e.g. Ca: 305 outliers, K: 19 outliers) before PCA is applied.

## Limitations

- IonFlow requires the pre-processing section to run first as it produces the cleaned dataset used by all other sections, including PCA.
- No quantitative benchmark results or performance metrics are reported for the two case studies (S. cerevisiae and HeLa ionomes).
- No explicit limitations or caveats are discussed in the paper's discussion section.
- Extreme outliers (points beyond Q1-3*IQ / Q3+3*IQ fences, e.g. up to 0.22% for Ca) can distort variance structure if not removed prior to PCA.

## Evidence

- [abstract] IonFlow pipeline including PCA step: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis"
- [full_text] IonAnalysis PCA on z-score profiles: [evidence span withheld — anchor status: not_found]
- [intro] IonFlow designed for ICP-MS ionome datasets: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
- [readme] IonFlow R package sections: [evidence span withheld — anchor status: external_doc]
- [discussion] IonFlow tested on yeast and HeLa ionomes: "the pipeline was tested on two large benchmark ionome datasets, the ionome of haploid S.Cervisieae and the ionome of HeLa human cells"
