---
name: sample-grouping-annotation-plotting
description: Use when you have a vsn-normalized MultiAssayExperiment (MetaboDiff 'met'
  object) and need an unsupervised check of whether sample groups separate metabolome-wide,
  before any differential testing.
when_to_use_negative:
- The goal is supervised differential metabolomic analysis (Student's T-Test/ANOVA
  with Benjamini-Hochberg correction) — use MetaboDiff's differential testing functions
  instead of unsupervised plots
- The measurements are raw and not yet variance stabilizing normalized (vsn); normalize
  first so PCA/tSNE distances are comparable across metabolites
- The aim is exploring sample traits in the data-derived metabolic correlation network
  — that is MetaboDiff's separate key functionality, not this pairwise-group visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboDiff
  role: Provides pca_plot and tsne_plot, the two unsupervised visualization functions
    for the metabolome-wide difference between samples, plus the tutorial MultiAssayExperiment
    object
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment; the MetaboDiff R package requires R version 4.0.2 or
    higher
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Object class produced by create_mae that stores the normalized metabolite
    matrix (norm slot) consumed by pca_plot/tsne_plot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:14d3421db59d646c9febc91d493a4f177d47328a1a6e17e00c64bed31e583e2b
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# sample-grouping-annotation-plotting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates unsupervised, group-annotated visualizations (PCA and tSNE) of the metabolome-wide difference between sample groups using MetaboDiff's pca_plot and tsne_plot, colored by a sample annotation factor such as 'tumor_normal'. This is the MetaboDiff tutorial's first-pass check for whether sample traits separate globally before differential testing.

## When to use

Use when you have a vsn-normalized MultiAssayExperiment (MetaboDiff 'met' object) and need an unsupervised check of whether sample groups separate metabolome-wide, before any differential testing.

## When NOT to use

- The goal is supervised differential metabolomic analysis (Student's T-Test/ANOVA with Benjamini-Hochberg correction) — use MetaboDiff's differential testing functions instead of unsupervised plots
- The measurements are raw and not yet variance stabilizing normalized (vsn); normalize first so PCA/tSNE distances are comparable across metabolites
- The aim is exploring sample traits in the data-derived metabolic correlation network — that is MetaboDiff's separate key functionality, not this pairwise-group visualization

## Inputs

- MetaboDiff MultiAssayExperiment object ('met') with a populated norm slot (vsn-normalized metabolite measurement table)
- Sample metadata annotation column, e.g. 'tumor_normal', present in the object's colData

## Outputs

- PCA plot of the metabolome-wide difference colored by group_factor (PCA_plot.png)
- tSNE plot colored by the same group_factor (tSNE_plot.png)
- Side-by-side figure used to visually inspect group separation along the first two principal components

## How to apply

Extract the normalized measurements from the norm slot of the MultiAssayExperiment object created by create_mae (e.g. the tutorial's Priolo et al. prostate cancer dataset of 61 tumors vs 25 normal tissues, profiled via Metabolon). Ensure variance stabilizing normalization (vsn) has been applied so variance is near-constant, then call MetaboDiff's pca_plot and tsne_plot with group_factor set to the annotation column of interest (e.g. 'tumor_normal') and label_colors mapped per group (e.g. c('darkseagreen','dodgerblue') for tumor vs normal). Save each plot (e.g. PCA_plot.png, tSNE_plot.png) and arrange them side by side with the tutorial's multiplot helper. Rationale: these two functions are the package's designated unsupervised explorations of the metabolome-wide difference; if the first two principal components do not separate tumor (T) from normal (N), the tutorial treats this as an expected observation and motivation for the downstream correlation-network and differential analyses rather than a failure.

## Related tools

- **MetaboDiff** (Provides pca_plot and tsne_plot, the two unsupervised visualization functions for the metabolome-wide difference between samples, plus the tutorial MultiAssayExperiment object) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment; the MetaboDiff R package requires R version 4.0.2 or higher) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Object class produced by create_mae that stores the normalized metabolite matrix (norm slot) consumed by pca_plot/tsne_plot)

## Examples

```
pca_plot(met, group_factor="tumor_normal", label_colors=c("darkseagreen","dodgerblue"))
```

## Evaluation signals

- Both output files (PCA_plot.png and tSNE_plot.png) exist and were produced with group_factor='tumor_normal'
- PCA plot shows no distinct metabolome-wide separation of tumor (T) vs normal (N) samples along the first two principal components, matching the tutorial's reported observation
- tSNE plot reproduces the same qualitative absence of group separation as the PCA
- Input matrix was taken from the norm slot (vsn-normalized), not raw measurements
- Label colors correctly map to the two groups (darkseagreen for one, dodgerblue for the other) as in the tutorial call

## Limitations

- Absence of separation along the first two PCs does not preclude group differences in individual metabolites — the tutorial proceeds to differential testing (t-test/ANOVA with Benjamini-Hochberg correction) regardless
- The functions are unsupervised and univariate in coloring only; they cannot quantify significance of any apparent clustering
- No changelog is available for the package ('No changelog found'), so version-specific behavior changes cannot be traced from the repository

## Evidence
