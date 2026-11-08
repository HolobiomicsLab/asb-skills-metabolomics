---
name: breast-cancer-phenotype-labeling
description: Use when a pathway deregulation score (PDS) matrix must be reduced to
  phenotype-informative features before classification, using a metabolomics table
  with a binary disease Label column.
when_to_use_negative:
- Input is still a raw metabolite-by-sample measurement matrix that has not been transformed
  to pathway deregulation scores by lilikoi.get_pd_scores.
- The data table lacks a phenotype Label column, so features cannot be scored against
  disease class.
- The goal is group-level pathway enrichment rather than building individual-sample
  (personalized) classification features.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: lilikoi
  role: Provides lilikoi.select_pathways, the feature selection call that scores pathway
    features against the phenotype Label
  repo: https://github.com/lanagarmire/lilikoi
- name: RWeka
  role: Supplies the information-gain (mutual information) attribute evaluation used
    internally, with gain ratio as the alternative method
- name: R
  role: Runtime environment in which the lilikoi feature selection module executes
  repo: https://cran.r-project.org/web/packages/lilikoi/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:9cd1e3c59e54d489fa40d9d084a8c3581f15f2b5fed268b66dc4afb79be1c20b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - RWeka
  - lilikoi
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# breast-cancer-phenotype-labeling

## Summary

Labels and ranks pathway-level features by their relevance to breast cancer phenotype (the Label column of plasma_breast_cancer.csv) using information-gain attribute evaluation, so only phenotype-informative pathways from the PDS matrix enter classification.

## When to use

Use when a pathway deregulation score (PDS) matrix must be reduced to phenotype-informative features before classification, using a metabolomics table with a binary disease Label column.

## When NOT to use

- Input is still a raw metabolite-by-sample measurement matrix that has not been transformed to pathway deregulation scores by lilikoi.get_pd_scores.
- The data table lacks a phenotype Label column, so features cannot be scored against disease class.
- The goal is group-level pathway enrichment rather than building individual-sample (personalized) classification features.

## Inputs

- PDS score matrix (pathways x samples) from lilikoi.get_pd_scores
- metabolite measurement table plasma_breast_cancer.csv containing a phenotype Label column

## Outputs

- selected_pathways.csv: ranked table of pathway identifiers and information-gain scores (all > 0.42, in [0,1])
- significantPathways object passed to lilikoi.machine_learning

## How to apply

Load the PDS score matrix (pathways x samples) produced by lilikoi.get_pd_scores and the metabolite measurement table plasma_breast_cancer.csv, which carries the phenotype Label column. Call lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain"), which internally invokes RWeka's information-gain (mutual information) attribute evaluation to score each pathway feature against the sample labels; switch method to gain ratio when the table contains additional categorical variables such as Age or Race. Filter pathways to those whose gain score exceeds the 0.42 threshold and sort them in descending order of relevance (scores range 0 to 1). Write the ranked pathway-name/score table to selected_pathways.csv for downstream lilikoi.machine_learning.

## Related tools

- **lilikoi** (Provides lilikoi.select_pathways, the feature selection call that scores pathway features against the phenotype Label) — https://github.com/lanagarmire/lilikoi
- **RWeka** (Supplies the information-gain (mutual information) attribute evaluation used internally, with gain ratio as the alternative method)
- **R** (Runtime environment in which the lilikoi feature selection module executes) — https://cran.r-project.org/web/packages/lilikoi/index.html

## Examples

```
significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain")
```

## Evaluation signals

- The R script runs without error and selected_pathways.csv is written.
- selected_pathways.csv contains a pathway identifier column and a gain-score column.
- Every retained pathway has an information-gain score strictly above the 0.42 threshold.
- All gain scores fall within the [0, 1] relevance range and the table is sorted in descending order.
- Downstream lilikoi.machine_learning accepts the significantPathways output without subsetting errors.

## Limitations

- The 0.42 threshold is dataset-specific; it must be re-tuned for other metabolomics cohorts.
- Method choice matters: plain information gain is the default, but gain ratio is recommended when additional categorical variables (e.g., Age, Race) are present.
- The skill depends on upstream dimension transformation; ~20% of metabolites may fail to map to standard HMDB IDs, shrinking the pathway feature space.
- RWeka is a hard dependency of the feature selection module.

## Evidence

- [abstract] feature selection module, which helps to select the significant pathway features related to the disease phenotypes: "feature selection module, which helps to select the significant pathway features related to the disease phenotypes"
- [methods] select_pathways call with threshold and gain method: "significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements,
  threshold = 0.42, method = "gain")"
- [methods] plasma_breast_cancer.csv example data file: "filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")"
- [readme] README description of mutual-information pathway selection: "Lilikoi computes the pathway deregulation score for a given set of metabolites, selects the pathways with the highest mutual information and then uses them to build a classifier."
- [intro] feature selection module offering a range of algorithms: "feature selection, which provides the user with a range of feature selection algorithms to select significant features related to phenotypes"
