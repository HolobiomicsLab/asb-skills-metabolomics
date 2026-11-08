---
name: metabolomics-principal-component-analysis
description: Use when starting a differential metabolomic analysis and you need an
  unsupervised, label-aware overview of whether sample groups separate metabolome-wide,
  before committing to hypothesis tests.
when_to_use_negative:
- When the goal is supervised differential testing between groups — MetaboDiff applies
  a Student's T-Test (two groups) or ANOVA (>2 groups) with Benjamini-Hochberg correction
  instead of unsupervised plots
- When the metabolite table has not yet been imputed (k-NN with 40% missingness cutoff)
  and vsn-normalized — run the normalization workflow step first
- When the goal is exploring sample traits in the data-derived metabolic correlation
  network, MetaboDiff's stated key functionality rather than low-dimensional ordination
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboDiff
  role: Provides pca_plot and tsne_plot, the two unsupervised visualization functions
    for the metabolome-wide difference between samples
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment; MetaboDiff requires R version 4.0.2 or higher
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:314ac51651ba58544e202ee6a862b3fd8e0f1002642bba257c92ef581d458e04
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-principal-component-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unsupervised exploration of the metabolome-wide difference between samples using MetaboDiff's pca_plot and tsne_plot functions on vsn-normalized metabolite measurements, to visualize whether groups (e.g. tumor vs normal) separate in low-dimensional space.

## When to use

Use when starting a differential metabolomic analysis and you need an unsupervised, label-aware overview of whether sample groups separate metabolome-wide, before committing to hypothesis tests.

## When NOT to use

- When the goal is supervised differential testing between groups — MetaboDiff applies a Student's T-Test (two groups) or ANOVA (>2 groups) with Benjamini-Hochberg correction instead of unsupervised plots
- When the metabolite table has not yet been imputed (k-NN with 40% missingness cutoff) and vsn-normalized — run the normalization workflow step first
- When the goal is exploring sample traits in the data-derived metabolic correlation network, MetaboDiff's stated key functionality rather than low-dimensional ordination

## Inputs

- MetaboDiff MultiAssayExperiment object (e.g. tutorial object `met`) with a populated norm slot of vsn-normalized metabolite measurements
- Grouping factor column (e.g. 'tumor_normal') present in the object's colData/metadata

## Outputs

- PCA_plot.png — PCA scatter plot colored by group_factor
- tSNE_plot.png — t-SNE scatter plot colored by group_factor
- Side-by-side combined figure (via multiplot) for visual inspection of tumor (T) vs normal (N) separation

## How to apply

Load MetaboDiff (installed via install_github('andreasmock/MetaboDiff') in R >= 4.0.2) together with its preprocessed tutorial MultiAssayExperiment object `met` derived from the Priolo et al. prostate cancer tissue metabolome (61 tumors vs 25 normal specimens profiled by Metabolon). Extract the variance-stabilizing-normalized (vsn) metabolite measurements from the object's norm slot as input. Call pca_plot with group_factor='tumor_normal' and label_colors=c('darkseagreen','dodgerblue') to render the PCA, and tsne_plot with the same arguments for the t-SNE embedding; save both as PNGs (PCA_plot.png, tSNE_plot.png) and arrange them side by side with the tutorial's multiplot helper. Rationale: these two functions are MetaboDiff's unsupervised exploration layer, applied after normalization so the embedding reflects vsn-stabilized variance rather than raw measurement noise.

## Related tools

- **MetaboDiff** (Provides pca_plot and tsne_plot, the two unsupervised visualization functions for the metabolome-wide difference between samples) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment; MetaboDiff requires R version 4.0.2 or higher)

## Examples

```
pca_plot(met, group_factor='tumor_normal', label_colors=c('darkseagreen','dodgerblue')); tsne_plot(met, group_factor='tumor_normal', label_colors=c('darkseagreen','dodgerblue'))
```

## Evaluation signals

- PCA plot shows no distinct metabolome-wide separation between tumor (T) and normal (N) samples along the first two principal components, matching the tutorial's reported observation for the Priolo et al. prostate cancer dataset
- Both output files PCA_plot.png and tSNE_plot.png exist
- Both plots use group_factor='tumor_normal' and label_colors c('darkseagreen','dodgerblue') so group identity is visually encoded
- Input measurements originate from the norm slot, i.e. after variance stabilizing normalization (vsn), not raw abundances

## Limitations

- Result is dataset-dependent: absence of tumor/normal separation along the first two PCs in this tissue metabolome means PCA/tSNE alone cannot confirm group differences; supervised testing (T-Test/ANOVA + Benjamini-Hochberg) is still required
- t-SNE embeddings are stochastic and sensitive to parameters; use PCA as the primary separation readout
- The tutorial example uses the Metabolon-profiled Priolo et al. prostate cohort (61 tumor / 25 normal); separation behavior may differ for other cohorts

## Evidence

- [full_text] unsupervised visualization functions: [evidence span withheld — anchor status: not_found]
- [methods] cohort: [evidence span withheld — anchor status: not_found]
