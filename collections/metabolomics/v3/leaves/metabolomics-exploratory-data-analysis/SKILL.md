---
name: metabolomics-exploratory-data-analysis
description: Use when a metabolomics matrix plus sample metadata table (e.g. plasma_breast_cancer.csv
  with its demo metadata) needs dimension-reduction visualization or assessment of
  demographic confounder effects before classification.
when_to_use_negative:
- The goal is disease classification or prognosis prediction — use lilikoi.machine_learning
  or lilikoi.prognosis instead; this skill is only exploratory assessment
- The metabolite features must first be mapped to pathways (lilikoi.MetaTOpathway
  / lilikoi.PDSfun); SOV/PCA/t-SNE operate on metabolite and metadata tables, not
  pathway-level matrices
- Data are not yet preprocessed (normalized/imputed); Lilikoi v2.0 lists preprocessing
  as a separate module before exploratory analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Lilikoi v2.0
  role: Provides the lilikoi.explr function implementing PCA, t-SNE (M3C visualization
    routines), and ANOVA F-score-based SOV confounder analysis
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which lilikoi.explr is invoked and the plot images
    and SOV table are written to disk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:2b2b36316b86abb4a73852ba6f5d4419b32fed787e60388dde30e3a4ea3cfb7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-exploratory-data-analysis

## Summary

Exploratory analysis of metabolomics data using Lilikoi v2.0's lilikoi.explr function, which renders PCA and t-SNE dimension-reduction plots (via M3C visualization routines) and computes a source-of-variation (SOV) confounder table based on ANOVA F-scores over metadata variables.

## When to use

Use when a metabolomics matrix plus sample metadata table (e.g. plasma_breast_cancer.csv with its demo metadata) needs dimension-reduction visualization or assessment of demographic confounder effects before classification.

## When NOT to use

- The goal is disease classification or prognosis prediction — use lilikoi.machine_learning or lilikoi.prognosis instead; this skill is only exploratory assessment
- The metabolite features must first be mapped to pathways (lilikoi.MetaTOpathway / lilikoi.PDSfun); SOV/PCA/t-SNE operate on metabolite and metadata tables, not pathway-level matrices
- Data are not yet preprocessed (normalized/imputed); Lilikoi v2.0 lists preprocessing as a separate module before exploratory analysis

## Inputs

- Metabolomics abundance matrix (e.g. plasma_breast_cancer.csv loaded via lilikoi.Loaddata)
- Sample metadata / demographic data frame (e.g. dt$Metadata) with variables to test as sources of variation

## Outputs

- PCA plot image file of sample embeddings
- t-SNE plot image file of sample embeddings
- SOV result table containing ANOVA F-score entries for the tested metadata variables

## How to apply

Load the metabolomics dataset and its metadata table into R with lilikoi.Loaddata (e.g. the built-in plasma_breast_cancer.csv from the lilikoi package extdata). Call lilikoi.explr passing the metabolites data frame and the demographic/metadata data frame, setting the pca and tsne flags to TRUE to compute principal-component and t-SNE sample embeddings rendered with the M3C visualization routines. The function also computes ANOVA F-scores across the metadata variables to quantify source-of-variation (confounder) effects and assembles the SOV result table. Write the PCA and t-SNE plot images to disk and export the SOV table as a result file; this exploratory step sits alongside Lilikoi v2.0's preprocessing (normalization, imputation) before pathway-based classification.

## Related tools

- **Lilikoi v2.0** (Provides the lilikoi.explr function implementing PCA, t-SNE (M3C visualization routines), and ANOVA F-score-based SOV confounder analysis) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which lilikoi.explr is invoked and the plot images and SOV table are written to disk)

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")); Metadata <- dt$Metadata; dataSet <- dt$dataSet
```

## Evaluation signals

- PCA and t-SNE plot image files exist on disk after the lilikoi.explr run
- The SOV result table contains ANOVA F-score entries for each tested metadata variable (expert review of table completeness)
- Metadata and dataSet objects from lilikoi.Loaddata are non-empty and correctly paired sample-by-sample before analysis
- Both the pca and tsne flags were set to TRUE so both embedding types are produced, not just one

## Limitations

- SOV quantifies confounder effects only for metadata variables supplied in the demographic data frame; unrecorded confounders are not assessed
- t-SNE embeddings are stochastic, so plot reproducibility depends on the underlying M3C routine settings
- The exploratory module supports visualization and confounder assessment only; it does not itself select features or classify disease phenotypes

## Evidence

- [intro] exploratory tools include PCA, t-SNE and SOV: "Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis (e.g., principal component analysis"
- [abstract] Lilikoi v2.0 supports exploratory analysis: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [readme] plasma_breast_cancer dataset loading: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
