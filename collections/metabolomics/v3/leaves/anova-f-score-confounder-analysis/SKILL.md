---
name: anova-f-score-confounder-analysis
description: Use when a metabolomics dataset has sample metadata (e.g., demographics)
  and you need to assess which variables drive variation (SOV), or when exploratory
  PCA/t-SNE views are needed before classification.
when_to_use_negative:
- The goal is disease classification or prognosis prediction — those belong to Lilikoi's
  feature-selection/machine-learning or Cox-PH/Cox-nnet prognosis modules, not exploratory
  SOV analysis.
- No sample metadata/demographic variables are available to test as sources of variation.
- The data are already pathway-transformed (e.g., a PDS matrix from lilikoi.PDSfun)
  rather than raw metabolite-level measurements.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: Lilikoi v2.0
  role: provides the lilikoi.explr function that performs SOV/ANOVA F-score analysis
    and optionally generates PCA and t-SNE plots (pca/tsne flags TRUE)
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: programming environment in which the exploratory analysis module is executed
- name: M3C
  role: visualization routines used by lilikoi.explr to render the PCA and t-SNE dimension-reduction
    plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:ace4b845f58b452d083ee8045b79e43c1169e3e2a91cf2476203e67e7cb42065
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - M3C
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ANOVA F-score confounder analysis

## Summary

Exploratory source-of-variation (SOV) analysis from Lilikoi v2.0 that quantifies confounder effects in metabolomics data by computing ANOVA F-scores across metadata variables, optionally alongside PCA and t-SNE dimension-reduction plots rendered via the M3C visualization routines.

## When to use

Use when a metabolomics dataset has sample metadata (e.g., demographics) and you need to assess which variables drive variation (SOV), or when exploratory PCA/t-SNE views are needed before classification.

## When NOT to use

- The goal is disease classification or prognosis prediction — those belong to Lilikoi's feature-selection/machine-learning or Cox-PH/Cox-nnet prognosis modules, not exploratory SOV analysis.
- No sample metadata/demographic variables are available to test as sources of variation.
- The data are already pathway-transformed (e.g., a PDS matrix from lilikoi.PDSfun) rather than raw metabolite-level measurements.

## Inputs

- metabolomics expression/intensity data frame (e.g., dataSet from plasma_breast_cancer.csv)
- sample metadata/demographic data frame (e.g., Metadata returned by lilikoi.Loaddata)

## Outputs

- SOV result table of ANOVA F-scores across tested metadata variables
- PCA plot image file written to disk
- t-SNE plot image file written to disk

## How to apply

Load the metabolomics matrix and its metadata table into R (e.g., the built-in plasma_breast_cancer.csv via lilikoi.Loaddata, which returns Metadata and dataSet). Call lilikoi.explr with the user-provided metabolites data frame and demographic data frame; set the pca and tsne flags to TRUE to compute principal-component and t-SNE sample embeddings and render the dimension-reduction plots using the M3C visualization routines. In the same step, compute ANOVA F-scores across the metadata variables to quantify source-of-variation (confounder) effects and assemble the SOV result table. Write the PCA/t-SNE plot images to disk and save the SOV table as a result file, then verify plot files exist and that the SOV table contains F-score entries for every tested metadata variable.

## Related tools

- **Lilikoi v2.0** (provides the lilikoi.explr function that performs SOV/ANOVA F-score analysis and optionally generates PCA and t-SNE plots (pca/tsne flags TRUE)) — https://github.com/lanagarmire/lilikoi2
- **R** (programming environment in which the exploratory analysis module is executed)
- **M3C** (visualization routines used by lilikoi.explr to render the PCA and t-SNE dimension-reduction plots)

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package="lilikoi")); lilikoi.explr(dt$dataSet, dt$Metadata, pca=TRUE, tsne=TRUE)
```

## Evaluation signals

- PCA and t-SNE plot image files exist on disk after the run
- SOV result table contains ANOVA F-score entries for each tested metadata variable
- Expert review confirms table completeness (no missing metadata variables in the SOV output)
- Data loaded successfully via lilikoi.Loaddata (Metadata and dataSet both populated before explorr is called)

## Limitations

- SOV analysis is exploratory: ANOVA F-scores quantify variation explained by metadata variables but do not establish causal confounding.
- Assessment depends on the completeness of the user-provided demographic/metadata data frame; missing metadata variables cannot be scored.
- PCA and t-SNE outputs are optional and only produced when the pca and tsne flags are explicitly set to TRUE.

## Evidence

- [intro] exploratory data analysis (e.g., PCA, t-SNE analysis, and SOV analysis): [evidence span withheld — anchor status: not_found]
- [readme] readme exploratory analysis support: "Additionally, Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression."
- [readme] readme data loading example: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
