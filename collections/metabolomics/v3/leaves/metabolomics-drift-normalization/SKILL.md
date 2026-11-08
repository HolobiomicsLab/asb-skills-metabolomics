---
name: metabolomics-drift-normalization
description: 'Use when a metabolomics intensity matrix (rawdata: compounds × sample
  runs) shows drift/batch effects across runs with QC samples interleaved, and drift
  correction is needed before differential abundance testing.'
when_to_use_negative:
- 'Datasets far beyond the package''s tested scale may need care with multivariate
  external tools: SERFF''s Shiny web interface failed to complete on the untargeted
  FAME saliva dataset (618 runs × 8913 features), so SERFF is unsuitable at that scale'
- If multivariate (cross-metabolite) outlier handling is required — Metanorm treats
  outliers only at the feature level and provides no automated outlier-detection-and-removal
  pipeline
- 'Do not pick a method by maximizing DAM count or minimizing QC-sample RSD: the paper
  explicitly flags these as benchmarking pitfalls'
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metanorm
  role: R package implementing tGAM, rGAM, rLOESS, QC-RLSC and QC-RSC batchwise normalization
    with diagnostic plots and parallelization
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Backend for tGAM/rGAM thin plate regression spline fitting (REML) with QCcheck
    observation weighting
- name: fANCOVA
  role: Provides loess.as used by the rLOESS robust cross-validated LOESS method with
    Tukey biweight downweighting
- name: SERFF
  role: Multivariate normalization comparison method run via its Shiny web interface;
    failed on the large FAME dataset
- name: TIGER
  role: Multivariate normalization comparison method run via its R package
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:e182122ccb271e3f4dca0eb1b1614e1014db3fc29530c8a6199a76c2c9affee7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - R
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolomics-drift-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batchwise signal-drift normalization of metabolomics intensity matrices using outlier-robust spline methods (tGAM, rGAM, rLOESS) implemented in the Metanorm R package, alongside QC-RLSC and QC-RSC baselines. tGAM is the recommended default because it best recovers the underlying signal for downstream differential abundance testing.

## When to use

Use when a metabolomics intensity matrix (rawdata: compounds × sample runs) shows drift/batch effects across runs with QC samples interleaved, and drift correction is needed before differential abundance testing.

## When NOT to use

- Datasets far beyond the package's tested scale may need care with multivariate external tools: SERFF's Shiny web interface failed to complete on the untargeted FAME saliva dataset (618 runs × 8913 features), so SERFF is unsuitable at that scale
- If multivariate (cross-metabolite) outlier handling is required — Metanorm treats outliers only at the feature level and provides no automated outlier-detection-and-removal pipeline
- Do not pick a method by maximizing DAM count or minimizing QC-sample RSD: the paper explicitly flags these as benchmarking pitfalls

## Inputs

- Numerical intensity matrix rawdata (rows = compounds, columns = sample runs), log-transformed (base e) before normalization
- Vector batch giving the batch label of each sample run
- Vector metanorm.qc (or type) labeling each run as QC or other sample type
- Optional group labels for downstream two-sided Wilcoxon rank sum differential abundance tests (significance called at p < 0.05)

## Outputs

- Normalized intensity matrix per normalization method (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC)
- Per-compound pre- vs post-normalization intensity-vs-order diagnostic plots written to plotdir
- PCA score plots before and after normalization, labeled by batch
- Differential abundance results and pairwise DAM overlap percentages / p-value Spearman correlations across methods (e.g. bioheart_dam_overlap.csv)

## How to apply

Natural-log (base e) transform the intensity matrix, then run batchwise normalization with the metanorm() function per compound, passing the batch vector and sample-type vector (QC vs non-QC) and setting QCcheck = TRUE so the package verifies whether QC samples are representative of biological samples. Call with model = "tGAM" (REML-fitted thin plate regression splines via mgcv) as the default; rGAM and rLOESS (loess.as in fANCOVA with Tukey biweight downweighting) are faster alternatives, while model = "QC-RLSC" (loess degree 2 with LOOCV/GCV span selection, run with QConly = TRUE) and QC-RSC (smooth.spline) serve as QC-only baselines. Use plotdir to generate per-compound pre- vs post-normalization intensity-vs-order plots and plotPCA() before/after to verify that batch effects diminished. Rationale: outliers in QC samples deteriorate QC-based normalization and reduce downstream statistical power, so the robust methods jointly or selectively use QC and biological samples; note that a larger DAM count (as seen with raw or TIGER data) is NOT evidence of better normalization — assess agreement via pairwise DAM overlaps and p-value Spearman correlations instead.

## Related tools

- **Metanorm** (R package implementing tGAM, rGAM, rLOESS, QC-RLSC and QC-RSC batchwise normalization with diagnostic plots and parallelization) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (Backend for tGAM/rGAM thin plate regression spline fitting (REML) with QCcheck observation weighting)
- **fANCOVA** (Provides loess.as used by the rLOESS robust cross-validated LOESS method with Tukey biweight downweighting)
- **SERFF** (Multivariate normalization comparison method run via its Shiny web interface; failed on the large FAME dataset)
- **TIGER** (Multivariate normalization comparison method run via its R package)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanorm_diagnostics/"); plotPCA(normdat, type = batch)
```

## Evaluation signals

- tGAM/rGAM/rLOESS pairwise DAM overlap is high — 92% joint findings on BioHEART (1359 runs × 53 metabolites) and 79–87% on FAME — whereas disagreement with QC-RLSC (55–59% on FAME) or SERFF (29–41% on BioHEART) marks method-specific differences
- plotPCA before vs after normalization labeled by batch shows batch effects diminished after correction
- Per-compound pre- vs post-normalization intensity-vs-order plots (from plotdir) show drift removed in biological, not just QC, samples — QC-RLSC can look fine in PCA while drift remains in non-QC runs
- Unnormalized raw data yields the highest DAM count; a normalized matrix exceeding this would indicate overcorrection rather than better normalization
- R version >= 4.4.0 is required for the package to install

## Limitations

- Multiple testing correction was not applied in the source work (Wilcoxon tests used p < 0.05 uncorrected), though the authors note it is essential for downstream inference
- Automated outlier detection and removal prior to normalization is nontrivial and not provided; outlier handling is feature-level only, and multivariate outlier detection is not implemented
- tGAM/rGAM run with QCcheck = TRUE rely on QC samples being representative — unexpected discrepancies between QC and biological samples require analyst scrutiny of the diagnostic plots
- SERFF (web Shiny interface) exceeded capacity on the large untargeted FAME dataset, so external multivariate-method comparison may be infeasible at scale

## Evidence

- [readme] README method inventory: [evidence span withheld — anchor status: external_doc]
- [readme] README recommended usage: [evidence span withheld — anchor status: external_doc]
