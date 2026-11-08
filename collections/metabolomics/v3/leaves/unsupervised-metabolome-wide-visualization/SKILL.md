---
name: unsupervised-metabolome-wide-visualization
description: Use when starting differential metabolomic analysis on a metabolite measurement
  table and you need an unsupervised check for metabolome-wide group separation before
  supervised testing.
when_to_use_negative:
- The goal is supervised differential testing between groups (t-test/ANOVA with Benjamini-Hochberg
  correction) rather than unsupervised visualization
- The analysis targets sample traits in the data-derived metabolic correlation network
  (WGCNA-based), which is a separate MetaboDiff key functionality
- Input is still raw, missing-value-laden measurement data that first requires k-nearest
  neighbor imputation (40% cutoff) and vsn normalization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDiff
  role: Provides the pca_plot and tsne_plot unsupervised visualization functions applied
    to the normalized MultiAssayExperiment object
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment for MetaboDiff (version 4.0.2 or higher) in which the
    plots are generated and saved as PNG
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:ee97b9663fbe5334eadad591fa4b496ffeeaa9b29aa06bc6ce972e7323890a42
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# unsupervised-metabolome-wide-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate unsupervised metabolome-wide visualizations (PCA and t-SNE) of normalized metabolite measurements using MetaboDiff's pca_plot and tsne_plot functions to check whether sample groups, e.g. tumor vs normal, separate along the first principal components.

## When to use

Use when starting differential metabolomic analysis on a metabolite measurement table and you need an unsupervised check for metabolome-wide group separation before supervised testing.

## When NOT to use

- The goal is supervised differential testing between groups (t-test/ANOVA with Benjamini-Hochberg correction) rather than unsupervised visualization
- The analysis targets sample traits in the data-derived metabolic correlation network (WGCNA-based), which is a separate MetaboDiff key functionality
- Input is still raw, missing-value-laden measurement data that first requires k-nearest neighbor imputation (40% cutoff) and vsn normalization

## Inputs

- MetaboDiff MultiAssayExperiment object (e.g. tutorial object 'met' from the Priolo et al. prostate cancer tissue metabolome)
- Normalized (vsn) metabolite measurements from the object's norm slot
- Sample metadata grouping factor (e.g. 'tumor_normal') for coloring labels

## Outputs

- PCA plot image (e.g. PCA_plot.png) colored by group_factor
- t-SNE plot image (e.g. tSNE_plot.png) colored by group_factor
- Side-by-side arrangement of the two plots for visual inspection of group separation

## How to apply

Load the MetaboDiff package and its preprocessed tutorial MultiAssayExperiment object (e.g. the Priolo et al. prostate cancer dataset, 61 tumors vs 25 normal specimens) and take the variance-stabilizing-normalized (vsn) measurements from the norm slot as input, since vsn is applied to keep the variance nearly constant. Call pca_plot with group_factor set to the grouping column (e.g. 'tumor_normal') and label_colors such as c('darkseagreen','dodgerblue'), then call tsne_plot with the same arguments, saving each as a PNG. Arrange both plots side by side (e.g. with the multiplot helper from the MetaboDiff tutorial) and inspect whether the first two principal components separate the groups. The rationale is that these two functions are MetaboDiff's designated unsupervised exploration of the metabolome-wide difference; the tutorial reports no distinct tumor-vs-normal separation along the first two principal components, which serves as the expected baseline observation.

## Related tools

- **MetaboDiff** (Provides the pca_plot and tsne_plot unsupervised visualization functions applied to the normalized MultiAssayExperiment object) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment for MetaboDiff (version 4.0.2 or higher) in which the plots are generated and saved as PNG)

## Examples

```
pca_plot(met, group_factor = "tumor_normal", label_colors = c("darkseagreen", "dodgerblue"))
```

## Evaluation signals

- Both PCA_plot.png and tSNE_plot.png files exist after the run
- PCA plot shows no distinct metabolome-wide separation between tumor (T) and normal (N) samples along the first two principal components, matching the tutorial's reported observation
- Points in both plots are colored by the group_factor using the specified label_colors c('darkseagreen','dodgerblue')
- Input measurements come from the vsn-normalized norm slot of the MultiAssayExperiment object, not raw intensities

## Limitations

- The unsupervised plots provide only a qualitative visual check; absence of tumor-vs-normal separation along the first two PCs does not rule out group differences detectable by supervised testing
- Requires prior normalization (vsn) and imputation steps; the tutorial pipeline applies k-means clustering with remove_cluster (cluster 1 removed, k=2) before downstream analysis
- The package documentation notes 'No changelog found.' in the discussion, so function interfaces may change without a tracked history

## Evidence

- [other] MetaboDiff provides two unsupervised visualization functions for the metabolome-wide difference between samples: pca_plot (principal component analysis) and tsne_plot (t-distributed stochastic neighbor embedding).: [evidence span withheld — anchor status: not_found]
