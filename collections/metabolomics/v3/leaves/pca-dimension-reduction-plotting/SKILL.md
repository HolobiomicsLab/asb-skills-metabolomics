---
name: pca-dimension-reduction-plotting
description: Use when a metabolomics data frame plus demographic/metadata table is
  loaded in R and you need unsupervised sample-embedding plots (PCA, t-SNE) or a confounder/source-of-variation
  assessment before classification.
when_to_use_negative:
- The goal is supervised classification or prognosis prediction (lilikoi.machine_learning
  or lilikoi.prognosis), not unsupervised exploratory visualization
- The data have already been transformed to pathway-level features (PDS matrix) for
  pathway regression or KEGG plotting
- No demographic/metadata table is available, so the SOV confounder assessment cannot
  be computed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3913
tools:
- name: Lilikoi v2.0
  role: Provides the lilikoi.explr function that runs PCA/t-SNE plotting and SOV ANOVA
    analysis on metabolomics data
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which lilikoi.explr is invoked on the loaded data
    and metadata frames
- name: M3C
  role: Visualization routines used by lilikoi.explr to render the PCA and t-SNE dimension-reduction
    plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:5f5989349e57506d19d65f3fc1ceff972c43d85e35254ba2b2fff28eb9e8e910
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - M3C
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3913
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pca-dimension-reduction-plotting

## Summary

Lilikoi v2.0's exploratory analysis step that renders PCA and t-SNE dimension-reduction plots of metabolomics samples (via M3C visualization routines) and computes a source-of-variation (SOV) table of ANOVA F-scores across metadata variables to flag confounders.

## When to use

Use when a metabolomics data frame plus demographic/metadata table is loaded in R and you need unsupervised sample-embedding plots (PCA, t-SNE) or a confounder/source-of-variation assessment before classification.

## When NOT to use

- The goal is supervised classification or prognosis prediction (lilikoi.machine_learning or lilikoi.prognosis), not unsupervised exploratory visualization
- The data have already been transformed to pathway-level features (PDS matrix) for pathway regression or KEGG plotting
- No demographic/metadata table is available, so the SOV confounder assessment cannot be computed

## Inputs

- Metabolomics expression data frame loaded via lilikoi.Loaddata (e.g., plasma_breast_cancer.csv from package extdata)
- Metadata table containing sample labels and demographic/clinical variables for SOV assessment
- Boolean flags pca and tsne indicating which dimension-reduction plots to generate

## Outputs

- PCA plot image file written to disk
- t-SNE plot image file written to disk
- SOV result table containing ANOVA F-scores for each tested metadata variable

## How to apply

Load the metabolite expression matrix and its Metadata table into R (e.g., with lilikoi.Loaddata on the built-in plasma_breast_cancer.csv from the lilikoi package extdata). Call lilikoi.explr with the metabolite data frame and the demographic data frame, setting the pca and tsne flags to TRUE so principal component and t-SNE sample embeddings are computed and rendered using the M3C visualization routines. In the same call, ANOVA F-scores are computed across each metadata variable to quantify source-of-variation (confounder) effects, assembling the SOV result table. Write the PCA and t-SNE plot images to disk and save the SOV table as a result file; inspect the plots and table to decide whether metadata variables (e.g., batch or demographic factors) dominate the embedding structure before proceeding to pathway transformation and classification.

## Related tools

- **Lilikoi v2.0** (Provides the lilikoi.explr function that runs PCA/t-SNE plotting and SOV ANOVA analysis on metabolomics data) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which lilikoi.explr is invoked on the loaded data and metadata frames)
- **M3C** (Visualization routines used by lilikoi.explr to render the PCA and t-SNE dimension-reduction plots)

## Evaluation signals

- The PCA and t-SNE plot image files exist on disk after the run
- The SOV result table contains ANOVA F-score entries for every tested metadata variable (expert review of table completeness)
- The generated PCA and t-SNE embeddings cover the same sample set as the loaded Metadata table (row correspondence check)
- Plots render without errors through the M3C visualization routines
- Metadata variables with high ANOVA F-scores visibly align with structure in the PCA/t-SNE embeddings, confirming the confounder signal is interpretable

## Limitations

- SOV assessment depends entirely on the completeness and correctness of the user-provided demographic metadata table; unrecorded confounders will not be flagged
- PCA and t-SNE are unsupervised, so separated or overlapping embeddings are exploratory evidence only and do not validate classification performance
- The exploratory module operates on preprocessed metabolite-level data; unnormalized or unimputed inputs can distort embeddings and F-scores

## Evidence

- [intro] Exploratory data analysis scope of Lilikoi v2.0: [evidence span withheld — anchor status: not_found]
- [readme] README module summary: "Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression."
