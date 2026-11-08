---
name: gcxgc-metabolomics-multivariate-analysis
description: Use when GCxGC-MS 2D-TIC chromatograms (e.g., NetCDF imports or bundled
  MTBLS579 data) have been preprocessed and you need unsupervised multivariate analysis
  scores/loadings to distinguish sample groups.
when_to_use_negative:
- You need supervised classification/regression between groups — instead export chromatograms
  with unfold_chrom and use mixOmics models, as the package recommends for supervised
  analysis
- Data are one-dimensional chromatograms or already-unfolded feature tables rather
  than 2D-TIC GCxGC chromatograms handled by MPCA
- Raw, unpreprocessed chromatograms with noise/baseline misalignment — the package
  workflow requires smoothing, baseline correction, and peak alignment before multivariate
  analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: RGCxGC
  role: Provides m_prcomp for fitting MPCA, scores() for extracting the score matrix,
    plot_loading for loading visualization, and read_chrom for NetCDF import
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Runtime environment for executing the RGCxGC pipeline and exporting scores/loadings
    to CSV
- name: mixOmics
  role: Optional downstream supervised modeling on chromatograms exported via unfold_chrom
  repo: https://bioconductor.org/packages/release/bioc/html/mixOmics.html
- name: MTBLS579
  role: Example dataset (MTBLS.rda) bundled for reproducing the MPCA workflow
  repo: https://github.com/DanielQuiroz97/MTBLS579
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:7ba0ddb360785ea1cd79abe511c1440273e96fdc872ffce139cee59cb520a7b6
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
  - mixOmics
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# gcxgc-metabolomics-multivariate-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit multiway principal component analysis (MPCA, Wold's approach) on preprocessed GCxGC-MS two-dimensional chromatograms with the RGCxGC package's m_prcomp, then extract the scores matrix via scores() and loadings via plot_loading or direct export. This reveals the main variables distinguishing sample groups after smoothing, baseline correction, and 2D-COW alignment.

## When to use

Use when GCxGC-MS 2D-TIC chromatograms (e.g., NetCDF imports or bundled MTBLS579 data) have been preprocessed and you need unsupervised multivariate analysis scores/loadings to distinguish sample groups.

## When NOT to use

- You need supervised classification/regression between groups — instead export chromatograms with unfold_chrom and use mixOmics models, as the package recommends for supervised analysis
- Data are one-dimensional chromatograms or already-unfolded feature tables rather than 2D-TIC GCxGC chromatograms handled by MPCA
- Raw, unpreprocessed chromatograms with noise/baseline misalignment — the package workflow requires smoothing, baseline correction, and peak alignment before multivariate analysis

## Inputs

- Preprocessed two-dimensional chromatogram object (2D-TIC) from RGCxGC, e.g., the bundled MTBLS.rda / MTBLS579 dataset
- Raw NetCDF GCxGC-MS chromatogram files imported via read_chrom (with mod_time, sam_rate, per_eval parameters)

## Outputs

- MPCA scores matrix (samples x retained principal components), exportable as mpca_scores.csv
- Principal component loading values (per second-dimension retention-time slice), exportable as mpca_pc1_loadings.csv or rendered via plot_loading (type 'n'/'p')
- Model summary from the fitted m_prcomp object

## How to apply

After importing the raw NetCDF chromatogram and folding it into the 2D-TIC, and applying smoothing, baseline correction (asymmetric least squares), and 2D-COW peak alignment, fit MPCA with RGCxGC's m_prcomp on the chromatogram object, using center = TRUE, scale = FALSE as in the package vignette (optionally limiting npcs). Retrieve the score matrix with the scores() accessor — one row per sample — and export it (e.g., mpca_scores.csv). Extract loading values for a chosen principal component and plot them with plot_loading using type = 'n' for negative loadings or type = 'p' for positive loadings, or export the first-component loadings (one per second-dimension retention-time slice) to mpca_pc1_loadings.csv. Validate that score dimensions match the number of samples and retained components of the fitted m_prcomp object. Because MPCA handles the three dimensions typical of 2D-TIC GCxGC data, no manual unfolding of the chromatogram is required for this unsupervised step — unfold_chrom is reserved for exporting to supervised mixOmics models.

## Related tools

- **RGCxGC** (Provides m_prcomp for fitting MPCA, scores() for extracting the score matrix, plot_loading for loading visualization, and read_chrom for NetCDF import) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Runtime environment for executing the RGCxGC pipeline and exporting scores/loadings to CSV)
- **mixOmics** (Optional downstream supervised modeling on chromatograms exported via unfold_chrom) — https://bioconductor.org/packages/release/bioc/html/mixOmics.html
- **MTBLS579** (Example dataset (MTBLS.rda) bundled for reproducing the MPCA workflow) — https://github.com/DanielQuiroz97/MTBLS579

## Examples

```
library(RGCxGC); data(MTBLS); exp_MPCA <- m_prcomp(MTBLS, center = TRUE, scale = FALSE); write.csv(scores(exp_MPCA), 'mpca_scores.csv'); plot_loading(exp_MPCA, pc = 1, type = 'p')
```

## Evaluation signals

- Both mpca_scores.csv and mpca_pc1_loadings.csv exist after the run
- Scores CSV row structure: one row per observation (sample) — dimensionality equals number of samples x retained components of the fitted m_prcomp object
- Loadings CSV row structure: one row per second-dimension retention-time slice/column of the 2D chromatogram
- m_prcomp was called with the vignette parameters center = TRUE, scale = FALSE on the chromatogram object
- Loading values match what plot_loading renders as type 'n' (negative) and type 'p' (positive) for the chosen principal component

## Limitations

- MPCA is unsupervised — to reveal group differences under supervision, RGCxGC instead exports unfolded chromatograms for mixOmics models
- Multivariate analysis presupposes completed preprocessing (noise reduction via smoothing, baseline correction, peak alignment); applying m_prcomp to noisy raw 2D-TIC undermines the results
- No changelog found for the package, so version-specific behavior changes may be undocumented

## Evidence
