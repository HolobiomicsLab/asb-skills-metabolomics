---
name: metabolomics-tsne-dimension-reduction
description: Use when starting a differential metabolomic analysis in MetaboDiff and
  you need to check, before group comparisons, whether samples (e.g. tumor vs normal)
  separate metabolome-wide along the first two principal components.
when_to_use_negative:
- The goal is supervised differential testing between groups — use MetaboDiff's T-Test/ANOVA
  workflow with Benjamini-Hochberg correction instead of unsupervised plots
- The input is a raw metabolite measurement table that has not yet undergone imputation
  (kNN with 40% cutoff), outlier screening, and variance stabilizing normalization
  (vsn)
- The analysis goal is exploration of sample traits in the data-derived metabolic
  correlation network (MetaboDiff's WGCNA-based key functionality), not global sample-space
  visualization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboDiff
  role: R package providing the pca_plot and tsne_plot unsupervised visualization
    functions applied to the vsn-normalized measurements in the MultiAssayExperiment
    norm slot
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment for MetaboDiff (version 4.0.2 or higher) in which the
    tutorial dataset is loaded and the plots are generated and saved
- name: MultiAssayExperiment
  role: Object container (created by create_mae) holding the normalized metabolite
    measurements in the norm slot that serve as unsupervised-analysis input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:0c04b642ed7d338c1308cd5e2bbf6028cbb38e4d3f6da8c2cde398ea149f6bbc
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
  - MultiAssayExperiment
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolomics-tsne-dimension-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unsupervised exploration of the metabolome-wide difference between samples using MetaboDiff's pca_plot (PCA) and tsne_plot (t-SNE) visualization functions, applied to the normalized (vsn) metabolite measurements stored in a MultiAssayExperiment object.

## When to use

Use when starting a differential metabolomic analysis in MetaboDiff and you need to check, before group comparisons, whether samples (e.g. tumor vs normal) separate metabolome-wide along the first two principal components.

## When NOT to use

- The goal is supervised differential testing between groups — use MetaboDiff's T-Test/ANOVA workflow with Benjamini-Hochberg correction instead of unsupervised plots
- The input is a raw metabolite measurement table that has not yet undergone imputation (kNN with 40% cutoff), outlier screening, and variance stabilizing normalization (vsn)
- The analysis goal is exploration of sample traits in the data-derived metabolic correlation network (MetaboDiff's WGCNA-based key functionality), not global sample-space visualization

## Inputs

- MetaboDiff MultiAssayExperiment object (e.g. tutorial object 'met' from the Priolo et al. prostate cancer tissue metabolome)
- Normalized (vsn) metabolite measurement matrix extracted from the object's norm slot

## Outputs

- PCA_plot.png — PCA scatter plot colored by group_factor (tumor_normal)
- tSNE_plot.png — t-SNE scatter plot colored by group_factor (tumor_normal)
- Visual assessment of whether tumor (T) and normal (N) samples separate along the first two principal components

## How to apply

Load the MetaboDiff package and its preprocessed tutorial example dataset — the MultiAssayExperiment object 'met' derived from the Priolo et al. prostate cancer tissue metabolome (61 prostate cancers vs 25 normal specimens profiled by Metabolon®). Extract the variance-stabilizing-normalized (vsn) metabolite measurements from the object's norm slot as input for the unsupervised analysis. Call MetaboDiff's pca_plot with group_factor='tumor_normal' and label_colors c('darkseagreen','dodgerblue') to generate the PCA plot, and tsne_plot with the same group_factor and label_colors for the t-SNE plot, saving each as a PNG. Arrange the two plots side by side (e.g. with the multiplot helper used in the tutorial) and inspect whether the first two principal components separate tumor (T) from normal (N) samples. The rationale is that these two functions are the package's unsupervised visualization entry point, run prior to differential testing (Student's T-Test / ANOVA with Benjamini-Hochberg correction) to contextualize metabolome-wide structure.

## Related tools

- **MetaboDiff** (R package providing the pca_plot and tsne_plot unsupervised visualization functions applied to the vsn-normalized measurements in the MultiAssayExperiment norm slot) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment for MetaboDiff (version 4.0.2 or higher) in which the tutorial dataset is loaded and the plots are generated and saved)
- **MultiAssayExperiment** (Object container (created by create_mae) holding the normalized metabolite measurements in the norm slot that serve as unsupervised-analysis input)

## Examples

```
pca_plot(met, group_factor='tumor_normal', label_colors=c('darkseagreen','dodgerblue')); tsne_plot(met, group_factor='tumor_normal', label_colors=c('darkseagreen','dodgerblue'))
```

## Evaluation signals

- PCA plot shows no distinct metabolome-wide separation between tumor and normal samples along the first two principal components, matching the tutorial's reported observation
- Both output files, PCA_plot.png and tSNE_plot.png, exist after execution
- Both plots are colored by group_factor='tumor_normal' using label_colors c('darkseagreen','dodgerblue')
- Input matrix is the vsn-normalized data from the MultiAssayExperiment norm slot, not raw or imputed-only measurements
- The two plots can be arranged side by side with the multiplot helper as in the tutorial

## Limitations

- In the tutorial example, PCA does not reveal a distinct metabolome-wide separation between tumor and normal samples along the first two principal components — absence of separation does not preclude group differences detectable by supervised testing
- The tutorial dataset is prostate tissue metabolome data from Metabolon® profiling (61 cancers, 25 normal specimens); group_factor and label_colors values must be adapted for other cohorts
- MetaboDiff requires R version 4.0.2 or higher

## Evidence

- [methods] pca_plot and tsne_plot as unsupervised visualization functions: [evidence span withheld — anchor status: not_found]
