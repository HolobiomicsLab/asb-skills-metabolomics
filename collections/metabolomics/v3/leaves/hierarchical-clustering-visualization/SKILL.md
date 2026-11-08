---
name: hierarchical-clustering-visualization
description: Use when you have a line-aggregated, z-score-standardised ionome profile
  (e.g. ICP-MS ion concentrations across yeast knockout mutants) and need to group
  elements by similarity and display element-element structure as a clustered heat
  map.
when_to_use_negative:
- Data are raw and have not passed the IonFlow PreProcessing section (outlier removal,
  log transform, median batch correction, standardisation) — clustering noisy, unbatch-corrected
  concentrations will be misleading
- The goal is sample/mutant-level clustering rather than element-level correlation
  structure; the described workflow clusters measured ions/elements
- Input is metabolomics feature data without element-concentration semantics — IonFlow's
  applicability to metabolomics is only noted as potential, not validated
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: IonFlow
  role: 'R package implementing the clustering section: Ward''s hierarchical clustering
    with Euclidean distance on z-score ion profiles, generating the correlation heat
    map'
  repo: https://github.com/wanchanglin/ionflow
- name: R
  role: Language in which IonFlow (and its ward.D clustering/heatmap code) is written
    and executed
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: User-facing platform through which the IonFlow clustering tool is run and
    configured (thres_ion_corr, method_ion_corr)
  repo: https://github.com/galaxyproject/galaxy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:3353b43789e790bbe53b6ec3b13c212361d811e5a9fb859a9966c24d3c18647f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hierarchical-clustering-visualization

## Summary

Hierarchical clustering of measured ions/elements from a pre-processed ionome dataset using Ward's minimum-variance method on Euclidean distances, visualised as a correlation heat map ordered by the resulting dendrogram. In IonFlow this is one of the exploratory-analysis stages alongside PCA and relevance network extraction.

## When to use

Use when you have a line-aggregated, z-score-standardised ionome profile (e.g. ICP-MS ion concentrations across yeast knockout mutants) and need to group elements by similarity and display element-element structure as a clustered heat map.

## When NOT to use

- Data are raw and have not passed the IonFlow PreProcessing section (outlier removal, log transform, median batch correction, standardisation) — clustering noisy, unbatch-corrected concentrations will be misleading
- The goal is sample/mutant-level clustering rather than element-level correlation structure; the described workflow clusters measured ions/elements
- Input is metabolomics feature data without element-concentration semantics — IonFlow's applicability to metabolomics is only noted as potential, not validated

## Inputs

- Line-aggregated z-score-standardised ion concentration profiles (pre-processed ICP-MS ionome dataset)
- Correlation threshold parameters (thres_ion_corr) and correlation method (method_ion_corr)
- Optional pre-defined ion standard deviations (e.g. computed from control strains such as YDL227C)

## Outputs

- Hierarchically clustered correlation heat map (dendrogram-ordered element-element correlation matrix)
- Element clustering that supports the relevance network (plot.net) output
- Dendrogram grouping of related elements

## How to apply

Within the IonFlow R package, take the line-aggregated z-score profiles produced by the PreProcessing section as the clustering input. Compute an element-element (dis)similarity using Euclidean distance on the ion profiles, then apply hierarchical clustering with Ward's method (ward.D) to group elements with similar behaviour across mutants. Couple the clustering with the element-element correlation matrix (thresholded by the user-specified thres_ion_corr and method_ion_corr parameters) so that the heat map cells report pairwise ion correlations, and order rows/columns by the dendrogram. The same clustering context feeds the relevance network (plot.net) where link width is proportional to the correlation coefficient. Rationale: Ward's linkage on Euclidean distance yields compact element groups, making correlated ion accumulations (e.g. among the 14 measured ions Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn) directly visible in the heat map.

## Related tools

- **IonFlow** (R package implementing the clustering section: Ward's hierarchical clustering with Euclidean distance on z-score ion profiles, generating the correlation heat map) — https://github.com/wanchanglin/ionflow
- **R** (Language in which IonFlow (and its ward.D clustering/heatmap code) is written and executed) — https://github.com/wanchanglin/ionflow
- **Galaxy** (User-facing platform through which the IonFlow clustering tool is run and configured (thres_ion_corr, method_ion_corr)) — https://github.com/galaxyproject/galaxy

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- The heat map rows/columns must be ordered according to a dendrogram derived from Euclidean distance with ward.D linkage, not alphabetical or arbitrary order
- Cell values in the heat map must equal the element-element correlation coefficients computed from the z-score profiles under the specified correlation method
- Cross-check with the relevance network: elements linked in the plot.net output (links whose correlation exceeds thres_ion_corr) should appear as adjacent, high-correlation blocks in the clustered heat map
- Link widths in the companion relevance network must be proportional to the correlation coefficients
- Clustering should reproduce the grouping of the 14 measured ions (Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn) on the S. cerevisiae benchmark dataset without erroring on the reported statistics

## Limitations

- Results depend on the user-chosen correlation threshold (thres_ion_corr) and correlation method (method_ion_corr); no default values or recommended cutoffs are stated in the article
- The article reports no quantitative benchmark or performance metrics for the clustering outputs on the S. cerevisiae and HeLa datasets
- Outlier structure in the raw data (up to ~22% extreme outliers per ion, e.g. Ca 305 of 9999 values) means clustering on un-pre-processed data is unreliable; pre-processing is mandatory first
- Supplementary material contents describing clustering details are referenced only by file size, limiting reproducibility of exact plotting settings

## Evidence

- [abstract] PCA, hierarchical clustering and heatmap workflow: "including principal components analysis, hierarchical clustering, relevance network extraction and analysis"
- [readme] Clustering is one of four IonFlow package sections: [evidence span withheld — anchor status: external_doc]
- [readme] Clustering section includes GO annotation/enrichment in MetaboFlow: [evidence span withheld — anchor status: external_doc]
- [readme] Pre-processing is a mandatory prerequisite for other sections: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark ion set and organism: [evidence span withheld — anchor status: external_doc]
