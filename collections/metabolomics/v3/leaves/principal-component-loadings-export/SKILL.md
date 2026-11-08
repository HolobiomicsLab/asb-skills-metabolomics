---
name: principal-component-loadings-export
description: Use when a fitted m_prcomp MPCA model on 2D-TIC GCxGC chromatograms (e.g.,
  MTBLS579) needs its scores matrix or PC loadings exported as numeric CSV files for
  downstream use or validation.
when_to_use_negative:
- The goal is supervised modeling rather than unsupervised MPCA extraction — RGCxGC
  exports unfolded chromatograms via unfold_chrom for mixOmics supervised models instead.
- Raw NetCDF chromatograms have not yet been imported and preprocessed (read_chrom,
  baseline correction, smoothing, 2D-COW alignment); MPCA is performed over preprocessed
  chromatograms.
- Only a visual rendering of loadings is needed and plot_loading suffices — no CSV
  export is required.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3810
- http://edamontology.org/topic_0121
tools:
- name: RGCxGC
  role: Provides m_prcomp to fit the MPCA, scores() to extract the score matrix, and
    plot_loading to render PC loadings ('n' numerical or 'p' points)
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Execution environment in which the RGCxGC extraction and CSV export scripts
    run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:2de56d11ab37941dab68f99226a433cebcdf9b2241a9b1435ff2177d951755ad
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
  - R
  - RGCxGC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3810
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# principal-component-loadings-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract and export the numerical results of multiway principal component analysis (MPCA) fitted with RGCxGC's m_prcomp: the scores matrix via scores(), and principal component loading values that underlie plot_loading, written out as CSV files.

## When to use

Use when a fitted m_prcomp MPCA model on 2D-TIC GCxGC chromatograms (e.g., MTBLS579) needs its scores matrix or PC loadings exported as numeric CSV files for downstream use or validation.

## When NOT to use

- The goal is supervised modeling rather than unsupervised MPCA extraction — RGCxGC exports unfolded chromatograms via unfold_chrom for mixOmics supervised models instead.
- Raw NetCDF chromatograms have not yet been imported and preprocessed (read_chrom, baseline correction, smoothing, 2D-COW alignment); MPCA is performed over preprocessed chromatograms.
- Only a visual rendering of loadings is needed and plot_loading suffices — no CSV export is required.

## Inputs

- Preprocessed two-dimensional chromatogram object (2D-TIC) accepted by m_prcomp, e.g., the bundled MTBLS.rda dataset from github.com/DanielQuiroz97/MTBLS579
- Fitted m_prcomp MPCA model object (center = TRUE, scale = FALSE)

## Outputs

- mpca_scores.csv — scores matrix, one row per sample/observation and one column per retained principal component
- mpca_pc1_loadings.csv — loading values for the chosen principal component (per second-dimension retention-time slice/column)

## How to apply

Load the preprocessed two-dimensional chromatogram data (the MTBLS.rda example bundle from the MTBLS579 repo) into the RGCxGC package environment, then fit multiway principal component analysis with m_prcomp using center = TRUE and scale = FALSE as in the package vignette. Retrieve the scores matrix with the scores() accessor and write it to mpca_scores.csv. Extract the loading matrix from the fitted model, retain the chosen principal component (e.g., PC1) loading values — the same values rendered by plot_loading with type = 'n' (numerical) or type = 'p' (points) for negative or positive loadings — and export them as mpca_pc1_loadings.csv. Validate that both CSVs exist, that scores have one row per observation with columns per retained component, and that loadings map to second-dimension retention-time slices consistent with the fitted model dimensions.

## Related tools

- **RGCxGC** (Provides m_prcomp to fit the MPCA, scores() to extract the score matrix, and plot_loading to render PC loadings ('n' numerical or 'p' points)) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Execution environment in which the RGCxGC extraction and CSV export scripts run)

## Examples

```
library(RGCxGC); data(MTBLS); exp_MPCA <- m_prcomp(MTBLS, center = TRUE, scale = FALSE); write.csv(scores(exp_MPCA), 'mpca_scores.csv')
```

## Evaluation signals

- Both mpca_scores.csv and mpca_pc1_loadings.csv exist and are non-empty after the run.
- Scores CSV row count equals the number of samples in the fitted m_prcomp object; column count equals retained components (npcs).
- Loadings CSV rows correspond to second-dimension retention-time slices/columns of the folded 2D-TIC.
- Loading values exported match the values plotted by plot_loading with type = 'n' and type = 'p' for the chosen principal component.
- m_prcomp was called with center = TRUE, scale = FALSE matching the vignette specification.

## Limitations

- MPCA in RGCxGC follows Wold's multiway PCA approach and handles the three dimensions typical of 2D-TIC from GCxGC; results are only meaningful on preprocessed (noise-reduced, aligned) chromatograms.
- plot_loading renders only a chosen principal component at a time; the exported loadings are per-PC slices, not the full loading matrix unless iterated per component.
- No changelog is documented for the package ('No changelog found'), so version-specific behavior of scores()/plot_loading should be checked against the installed CRAN or GitHub version.

## Evidence

    ]"
- [readme] README confirms Wold's multiway PCA basis: [evidence span withheld — anchor status: external_doc]
