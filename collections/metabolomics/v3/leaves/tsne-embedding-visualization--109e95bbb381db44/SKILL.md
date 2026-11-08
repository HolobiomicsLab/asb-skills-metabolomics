---
name: tsne-embedding-visualization
description: Use when you have a metabolomics feature matrix with sample metadata
  and need to visually assess sample clustering, batch effects, or confounder-driven
  structure before classification modeling.
when_to_use_negative:
- The metabolite names still need to be mapped to pathways (HMDB ids) — that is lilikoi.MetaTOpathway/lilikoi.PDSfun
  territory, not exploratory embedding
- The goal is supervised feature selection or classification (lilikoi.featuresSelection
  / lilikoi.machine_learning), not unsupervised sample-structure visualization
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: Lilikoi v2.0
  role: Provides lilikoi.explr, which performs PCA, t-SNE (via M3C routines), and
    SOV analysis on metabolomics data
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which Lilikoi v2.0's exploratory analysis module
    runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:93ebc080b574ebf6a6129418d2ab44d8eab51916dd01d77da91546d73cfa830a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tsne-embedding-visualization

## Summary

Exploratory-analysis skill from Lilikoi v2.0 that embeds metabolomics samples in 2D via PCA and t-SNE (rendered with M3C visualization routines) and optionally computes source-of-variation (SOV) confounder scores via ANOVA F-tests against metadata variables.

## When to use

Use when you have a metabolomics feature matrix with sample metadata and need to visually assess sample clustering, batch effects, or confounder-driven structure before classification modeling.

## When NOT to use

- The metabolite names still need to be mapped to pathways (HMDB ids) — that is lilikoi.MetaTOpathway/lilikoi.PDSfun territory, not exploratory embedding
- The goal is supervised feature selection or classification (lilikoi.featuresSelection / lilikoi.machine_learning), not unsupervised sample-structure visualization

## Inputs

- Metabolomics feature matrix loaded from a CSV such as plasma_breast_cancer.csv via lilikoi.Loaddata (returns dataSet and Metadata)
- Sample metadata/demographic data frame (Metadata table with variables such as Label and confounder columns)

## Outputs

- PCA sample-embedding plot image written to disk
- t-SNE sample-embedding plot image written to disk
- SOV result table containing ANOVA F-score entries for the tested metadata variables

## How to apply

Load the metabolomics dataset (e.g., the built-in plasma_breast_cancer.csv from Lilikoi's extdata) and its Metadata table with lilikoi.Loaddata. Call lilikoi.explr with the metabolite and demographic data frames, setting the pca and tsne flags to TRUE so that principal component and t-SNE sample embeddings are computed and rendered using the M3C visualization routines. In parallel, lilikoi.explr computes ANOVA F-scores across the metadata variables to quantify source-of-variation (SOV) confounder effects and assembles the SOV result table. Write the PCA/t-SNE plot images to disk and save the SOV table as a result file; inspect the plots for sample separation by phenotype label versus metadata variables to judge confounding.

## Related tools

- **Lilikoi v2.0** (Provides lilikoi.explr, which performs PCA, t-SNE (via M3C routines), and SOV analysis on metabolomics data) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which Lilikoi v2.0's exploratory analysis module runs)

## Examples

```
lilikoi.explr(dataSet, Metadata, pca=TRUE, tsne=TRUE)
```

## Evaluation signals

- PCA and t-SNE plot image files exist on disk after the lilikoi.explr run
- SOV result table is non-empty and contains ANOVA F-score entries for each tested metadata variable (expert review of table completeness)
- t-SNE embedding is 2D and renders without error through the M3C visualization routines
- Plots are consistent with the input dataset dimensions (one point per sample from plasma_breast_cancer.csv)

## Limitations

- SOV/confounder assessment depends on the quality and completeness of the user-provided metadata table; missing demographic variables yield incomplete ANOVA F-score entries
- The exploratory module is descriptive: PCA/t-SNE plots support visual inspection and do not by themselves constitute statistical hypothesis testing or pathway-level analysis

## Evidence

- [abstract] Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression.: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [intro] Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis (e.g., principal component analysis: "Last, we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis (e.g., principal component analysis"
- [readme] Additionally, Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression.: "Additionally, Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression."
