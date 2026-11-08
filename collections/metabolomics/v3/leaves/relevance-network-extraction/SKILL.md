---
name: relevance-network-extraction
description: Use when a pre-processed ionome dataset (line-aggregated z-score ion
  profiles from ICP-MS, e.g. 14 ions across yeast knockout mutants) needs element-element
  correlation network analysis.
when_to_use_negative:
- Input is raw ICP-MS concentration data that has not passed the IonFlow pre-processing
  section (outlier detection, log transform, median batch correction) — the README
  states pre-processing 'produces in output the cleaned dataset to be used in the
  other sections'
- The analysis goal is gene set enrichment / GO Slim annotation rather than element-element
  correlation network structure
- The data are not element (or analogous metabolite) concentration profiles, since
  IonFlow's relevance network is defined over measured ions in an ionome dataset
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: IonFlow
  role: R package whose IonAnalysis section computes element-element correlations
    and renders the plot.net relevance network given thres_ion_corr and method_ion_corr
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web platform hosting the IonFlow tool so the network extraction is accessible
    via a user-friendly interface on usegalaxy.org
  repo: https://github.com/galaxyproject/galaxy
- name: R
  role: Language in which IonFlow (and the underlying MetaboFlow IonFlow package)
    implements correlation computation and network plotting
  repo: https://github.com/AlinaPeluso/MetaboFlow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:3a73ca694d0d73516dbb90a1bd83cd9108bf8c1685e4ea0e632c80a7981b7be3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# relevance-network-extraction

## Summary

Extracts an element-element relevance network from a pre-processed ionome dataset by correlating ion profiles and linking pairs whose correlation exceeds a user-specified threshold, as implemented in the IonAnalysis section of the IonFlow Galaxy/R tool.

## When to use

Use when a pre-processed ionome dataset (line-aggregated z-score ion profiles from ICP-MS, e.g. 14 ions across yeast knockout mutants) needs element-element correlation network analysis.

## When NOT to use

- Input is raw ICP-MS concentration data that has not passed the IonFlow pre-processing section (outlier detection, log transform, median batch correction) — the README states pre-processing 'produces in output the cleaned dataset to be used in the other sections'
- The analysis goal is gene set enrichment / GO Slim annotation rather than element-element correlation network structure
- The data are not element (or analogous metabolite) concentration profiles, since IonFlow's relevance network is defined over measured ions in an ionome dataset

## Inputs

- Pre-processed ionome dataset: line-aggregated z-score ion concentration profiles (element x sample matrix from ICP-MS measurements)
- thres_ion_corr: correlation threshold for linking elements
- method_ion_corr: correlation method used to compute element-element correlations

## Outputs

- plot.net relevance network output: nodes = elements, link width proportional to element-element correlation coefficient, edges only where correlation exceeds threshold
- Element-element correlation matrix underlying the network
- Correlation heat map ordered by the resulting hierarchical clustering (ward.D, Euclidean distance)

## How to apply

Load the line-aggregated z-score profiles produced by the IonFlow pre-processing section, then specify the correlation threshold (thres_ion_corr) and correlation method (method_ion_corr) within IonFlow. Compute the element-element correlation matrix from the input ion profiles, keeping only element pairs whose correlation exceeds thres_ion_corr. Each node in the resulting plot.net output represents a measured element (e.g. Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn in the S. cerevisiae benchmark) and the width of each link is drawn proportional to the correlation coefficient between the two elements. The network is typically interpreted alongside hierarchical clustering (Ward's method with Euclidean distance) and the correlation heat map generated from the same correlation matrix. Note that IonFlow can run this analysis on raw data as well as directly on normalised and standardised profiles, but the pre-processing section must run first when starting from raw concentrations to remove extreme outliers (Q1 - 3*IQ / Q3 + 3*IQ fences) and perform median batch correction.

## Related tools

- **IonFlow** (R package whose IonAnalysis section computes element-element correlations and renders the plot.net relevance network given thres_ion_corr and method_ion_corr) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Web platform hosting the IonFlow tool so the network extraction is accessible via a user-friendly interface on usegalaxy.org) — https://github.com/galaxyproject/galaxy
- **R** (Language in which IonFlow (and the underlying MetaboFlow IonFlow package) implements correlation computation and network plotting) — https://github.com/AlinaPeluso/MetaboFlow

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow)
```

## Evaluation signals

- The plot.net output is produced with nodes corresponding exactly to the measured elements in the input z-score profile matrix (e.g. the 14 ions Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn in the S. cerevisiae benchmark)
- Every edge in the network connects a pair whose element-element correlation exceeds thres_ion_corr under method_ion_corr; no below-threshold edges appear
- Edge widths in the network are proportional to the correlation coefficients, so edge width ordering matches the correlation matrix ordering
- The network's element groupings are consistent with the ward.D hierarchical clustering / Euclidean distance heat map generated from the same correlation matrix
- No missing values remain in the input (README reports none for the yeast benchmark), so every element pair yields a defined correlation

## Limitations

- No quantitative benchmark results or performance metrics are reported for the two case studies (haploid S. cerevisiae and HeLa ionome), so network quality must be judged from the plots themselves
- The output depends on user-chosen thres_ion_corr and method_ion_corr; the article gives no default values or cutoff recommendations in the available text
- Correlation-based relevance networks indicate co-variation between element profiles, not causal or regulatory relationships

## Evidence

- [abstract] Relevance network extraction is a core IonFlow analysis component: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis"
- [discussion] Network is defined over element-element correlations with threshold-based linking: "IonFlow is very versatile and it can be used to process raw data as well as directly process normalised and standardised data for advanced analysis such as genetic network extraction"
- [readme] IonFlow is an R package wrapped in Galaxy for ionomics processing and analysis: "Galaxy tool for processing and analysis of ionomics data , based on the modification of R package"
- [readme] The MetaboFlow/IonFlow workflow contains a dedicated network analysis section: [evidence span withheld — anchor status: external_doc]
- [readme] Network analysis is one of the four named workflow sections: "* Clustering which also includes the GO Slim annotation and the GO terms enrichment
* Network analysis"
