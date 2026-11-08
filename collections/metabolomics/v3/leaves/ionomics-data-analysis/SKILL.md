---
name: ionomics-data-analysis
description: Use when ICP-MS ionome concentration data (raw, normalised, or line-aggregated
  z-score profiles) need exploratory multivariate analysis such as PCA, element-element
  correlation networks, or hierarchical clustering heat maps.
when_to_use_negative:
- The input is raw, unfiltered ionome data — the PreProcessing section (outlier detection
  via Q1-3*IQ / Q3+3*IQ outer fences, log transform, median batch correction) must
  run first to produce the cleaned dataset
- The data are not element-concentration ionome profiles from ICP-MS (or analogous
  metabolomics-style profiles), since the pipeline's similarity measures and ion-specific
  outlier handling assume element-concentration structure
- A standardised, reproducible pipeline is not required and ad-hoc single analyses
  (e.g. one scatter plot) suffice — IonFlow exists because no pipeline previously
  allowed standardised multivariate/network analysis of ionome datasets
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: IonFlow
  role: R package whose IonAnalysis module implements PCA, element-element correlation,
    ward.D clustering, and relevance network extraction; wrapped as a Galaxy tool
  repo: https://github.com/wanchanglin/ionflow
- name: MetaboFlow
  role: Base workflow repository from which the IonFlow R package (subdir IonFlow)
    is installed
  repo: https://github.com/AlinaPeluso/MetaboFlow
- name: Galaxy
  role: User-friendly platform through which the IonFlow tool is installed (via tool_conf.xml
    or Tool Shed) and run
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:200b44fe9505db0ad8e369b1e0c4803c0d798cff0cd0f19a2c34d08ff6dee42b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - MetaboFlow
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# ionomics-data-analysis

## Summary

Standardised multivariate and network analysis of ionome datasets from ICP-MS experiments using the IonFlow R package/Galaxy tool: PCA, element-element correlation, ward.D hierarchical clustering with heat map, and relevance network extraction.

## When to use

Use when ICP-MS ionome concentration data (raw, normalised, or line-aggregated z-score profiles) need exploratory multivariate analysis such as PCA, element-element correlation networks, or hierarchical clustering heat maps.

## When NOT to use

- The input is raw, unfiltered ionome data — the PreProcessing section (outlier detection via Q1-3*IQ / Q3+3*IQ outer fences, log transform, median batch correction) must run first to produce the cleaned dataset
- The data are not element-concentration ionome profiles from ICP-MS (or analogous metabolomics-style profiles), since the pipeline's similarity measures and ion-specific outlier handling assume element-concentration structure
- A standardised, reproducible pipeline is not required and ad-hoc single analyses (e.g. one scatter plot) suffice — IonFlow exists because no pipeline previously allowed standardised multivariate/network analysis of ionome datasets

## Inputs

- Line-aggregated z-score standardised ion concentration profiles (output of IonFlow PreProcessing), e.g. ICP-MS ppm concentrations for 14 ions (Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66) across single-gene knockout samples
- Correlation threshold parameters: thres_ion_corr and method_ion_corr
- Optionally, pre-defined ion standard deviations computed from control strains (e.g. YDL227C control)

## Outputs

- Relevance network (plot.net) of element-element correlations above the input threshold, with edge width proportional to the correlation coefficient
- Correlation heat map ordered by ward.D hierarchical clustering (Euclidean distance)
- PCA summary of the most significant variance components in the ionome data
- Element-element correlation matrix

## How to apply

Load the cleaned, line-aggregated z-score ion profiles produced by IonFlow's PreProcessing section into the IonAnalysis module. Run principal components analysis (PCA) to identify the most significant variance in the ionome data, specifying correlation thresholds (thres_ion_corr, method_ion_corr) for downstream steps. Compute element-element correlations from the input profiles using the chosen correlation method and threshold, then link pairs of elements whose correlation exceeds the threshold into a relevance network where nodes are elements and link width is proportional to the correlation coefficient (plot.net output). Perform hierarchical clustering with Ward's method (ward.D) on Euclidean distances to group related elements and generate the correlation heat map ordered by the resulting dendrogram. Optional ion-profile-specific similarity measures (Iacovacci et al., 2020) can be enabled by the user in place of default similarity measures.

## Related tools

- **IonFlow** (R package whose IonAnalysis module implements PCA, element-element correlation, ward.D clustering, and relevance network extraction; wrapped as a Galaxy tool) — https://github.com/wanchanglin/ionflow
- **MetaboFlow** (Base workflow repository from which the IonFlow R package (subdir IonFlow) is installed) — https://github.com/AlinaPeluso/MetaboFlow
- **Galaxy** (User-friendly platform through which the IonFlow tool is installed (via tool_conf.xml or Tool Shed) and run) — https://github.com/galaxyproject/galaxy

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- PCA output identifies components capturing the most significant variance in the ionome data before any network step runs
- Relevance network (plot.net) contains edges only between element pairs whose correlation exceeds thres_ion_corr under the chosen method_ion_corr, with edge widths proportional to correlation coefficient
- Correlation heat map dendrogram is reproducible when Ward's method (ward.D) with Euclidean distance is re-applied to the same correlation matrix
- Benchmark outputs reproduce on the haploid S. cerevisiae ionome (1454 single-gene haploid knockouts, 14 ions) and HeLa cell ionome test data shipped in test-data
- Median of median-batch-corrected log concentrations is 0 for each ion, confirming upstream standardisation is intact before analysis

## Limitations

- No quantitative benchmark results or performance metrics are reported for the two case studies (S. cerevisiae and HeLa ionomes); applicability is illustrated through outputs rather than validated metrics
- No version number is stated for IonFlow or Galaxy; the Galaxy tool is a modification of the AlinaPeluso/MetaboFlow R package, so results may depend on which fork/commit is installed
- Outlier and missing-value handling rely on the PreProcessing section (outer-fence rule Q1 - 3*IQ / Q3 + 3*IQ); results are sensitive to unreliable samples that survive pre-processing

## Evidence

- [abstract] IonFlow is a Galaxy tool written in R for ionomics data analysis: "IonFlow is a Galaxy tool written in R for ionomics data analysis"
- [abstract] Pipeline benchmarks: "The pipeline is described and tested on two benchmark data sets of the haploid S. Cerevisiae ionome and of the human HeLa cell ionome."
- [intro] Design motivation: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
- [discussion] Versatility of input data: "IonFlow is very versatile and it can be used to process raw data as well as directly process normalised and standardised data for advanced analysis such as"
- [readme] Package structure: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark data description: [evidence span withheld — anchor status: external_doc]
- [readme] Pre-processing rationale: [evidence span withheld — anchor status: external_doc]
