---
name: scores-and-loadings-matrix-extraction
description: Use when an MPCA model has been fitted with m_prcomp on preprocessed
  two-dimensional chromatogram data and the score matrix or principal-component loading
  values must be exported as numeric CSVs rather than viewed as plots.
when_to_use_negative:
- The goal is to inspect loadings visually only — plot_loading with type = 'n' or
  'p' already renders negative/positive loading values without exporting numerics.
- The chromatograms have not been preprocessed (smoothing, baseline correction via
  baseline_corr, and 2D-COW alignment) — MPCA on raw noisy 2D-TIC data will not yield
  meaningful scores/loadings.
- A supervised model is needed instead of unsupervised MPCA — unfold_chrom exports
  chromatograms for mixOmics supervised models, which is a different analysis path.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RGCxGC
  role: Provides m_prcomp for fitting MPCA, scores() accessor for the score matrix,
    and plot_loading for PC loadings
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Execution environment; the workflow is an R script using the RGCxGC package
    vignette conventions
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: MTBLS579
  role: Bundled example dataset (MTBLS.rda) of preprocessed GCxGC-MS chromatograms
    used to fit and validate the MPCA
  repo: https://github.com/DanielQuiroz97/MTBLS579
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:7aee3ae504d4b7d931c8a5165aceabf7f43b3319f8191d004eb81ab79910166b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/DanielQuiroz97/RGCxGC
  repo_url: https://github.com/DanielQuiroz97/RGCxGC
  tools:
  - MTBLS579
  - R
  - RGCxGC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# scores-and-loadings-matrix-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract the numerical results of a multiway principal component analysis (MPCA) fitted on GCxGC-MS 2D-TIC chromatograms with RGCxGC: retrieve the score matrix via scores() and export PC loadings underlying plot_loading renderings, as demonstrated on the bundled MTBLS579 dataset.

## When to use

Use when an MPCA model has been fitted with m_prcomp on preprocessed two-dimensional chromatogram data and the score matrix or principal-component loading values must be exported as numeric CSVs rather than viewed as plots.

## When NOT to use

- The goal is to inspect loadings visually only — plot_loading with type = 'n' or 'p' already renders negative/positive loading values without exporting numerics.
- The chromatograms have not been preprocessed (smoothing, baseline correction via baseline_corr, and 2D-COW alignment) — MPCA on raw noisy 2D-TIC data will not yield meaningful scores/loadings.
- A supervised model is needed instead of unsupervised MPCA — unfold_chrom exports chromatograms for mixOmics supervised models, which is a different analysis path.

## Inputs

- Preprocessed two-dimensional chromatogram object (2D-TIC) accepted by RGCxGC, e.g. the MTBLS.rda example data from github.com/DanielQuiroz97/MTBLS579
- Fitted MPCA model object returned by m_prcomp (e.g. with center = TRUE, scale = FALSE)
- Choice of principal component number whose loadings to extract (e.g. PC1)

## Outputs

- mpca_scores.csv: score matrix with one row per observation/sample and columns per retained component
- mpca_pc1_loadings.csv: loading values for the chosen principal component (values underlying plot_loading 'n'/'p' renderings)

## How to apply

Load the preprocessed two-dimensional chromatogram object (e.g., the bundled MTBLS.rda example data from the MTBLS579 repo) into the RGCxGC package environment, then fit multiway PCA with m_prcomp using center = TRUE and scale = FALSE as specified in the package vignette. Retrieve the score matrix with the scores() accessor on the fitted model object and write it to mpca_scores.csv. Extract the loading matrix from the fitted model and retain the first principal component loading values, exporting them as mpca_pc1_loadings.csv; these numeric loadings are exactly what plot_loading renders with type = 'n' or 'p' for negative or positive loading values of a chosen PC. Validate that score dimensions match the number of samples and retained components of the fitted m_prcomp object, and that loading rows correspond to second-dimension retention-time slices.

## Related tools

- **RGCxGC** (Provides m_prcomp for fitting MPCA, scores() accessor for the score matrix, and plot_loading for PC loadings) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Execution environment; the workflow is an R script using the RGCxGC package vignette conventions) — https://github.com/DanielQuiroz97/RGCxGC
- **MTBLS579** (Bundled example dataset (MTBLS.rda) of preprocessed GCxGC-MS chromatograms used to fit and validate the MPCA) — https://github.com/DanielQuiroz97/MTBLS579

## Examples

```
exp_MPCA <- m_prcomp(MTBLS_chrom, center = TRUE, scale = FALSE); write.csv(scores(exp_MPCA), "mpca_scores.csv"); write.csv(exp_MPCA$loadings[, 1], "mpca_pc1_loadings.csv")
```

## Evaluation signals

- mpca_scores.csv exists and its dimensions match the number of samples and the retained components (npcs) of the fitted m_prcomp object
- mpca_pc1_loadings.csv exists with one row per second-dimension retention-time slice/column
- Exported loading values reproduce what plot_loading(type = 'n') and plot_loading(type = 'p') display for the chosen PC (negative and positive values respectively)
- Scores/loadings are reproducible by re-running m_prcomp with the vignette parameters center = TRUE, scale = FALSE on the same MTBLS.rda object

## Limitations

- MPCA is Wold's multiway PCA approach and handles the three dimensions of 2D-TIC GCxGC data; loadings are indexed to retention-time slices, so any change in second-dimension slicing invalidates row correspondence
- Results depend on upstream preprocessing (asymmetric least squares baseline correction, Whittaker smoothing, 2D-COW peak alignment) — unpreprocessed chromatograms give unstable components

## Evidence

    ]"
