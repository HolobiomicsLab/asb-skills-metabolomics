---
name: qc-based-batch-correction
description: Use when a metabolomics intensity matrix (compounds × runs) spans multiple
  acquisition batches, QC samples are interspersed in the run order, and signal drift/batch
  effects must be removed before differential abundance testing.
when_to_use_negative:
- The matrix contains no interspersed QC samples — QC-RLSC/QC-RSC are unusable and
  the QC-vs-biological fit checks cannot run
- Multivariate (cross-metabolite) outlier detection is required — the article notes
  treatment is currently feature-level, and multivariate methods like SERFF and TIGER
  were not benchmarked within Metanorm
- Data are already normalized or the goal is only statistical inference — applying
  batch correction would alter p-values without addressing a drift problem (note multiple
  testing correction was explicitly out of scope here)
edam_operation: http://edamontology.org/operation_2939
tools:
- name: Metanorm R package
  role: 'Primary implementation: metanorm() performs batchwise QC-based correction
    with model = tGAM/rGAM/rLOESS/QC-RLSC/QC-RSC; plotPCA() and the plotdir diagnostics
    verify performance'
  repo: https://github.com/UGent-LIMET/Metanorm
- name: mgcv
  role: Fits the tGAM/rGAM normalization curves as (generalized) additive models with
    thin plate regression splines under REML
- name: fANCOVA
  role: Provides loess.as used for the cross-validated robust LOESS (rLOESS) fit with
    Tukey biweight downweighting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:5baa1621558b3b18607c6142f0672e5da745d3ef00db5bc693f4414f4ef9daff
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
  - Metanorm R package
  - R
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# qc-based-batch-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batchwise metabolomics normalization that fits intensity-vs-run-order curves (QC-based signal-drift correction) per compound, implemented in the Metanorm R package as QC-RLSC, QC-RSC, and the outlier-robust variants rLOESS, rGAM, and tGAM. The skill exists because QC-sample outliers deteriorate classical correction and inflate/deflate downstream differential abundance calls.

## When to use

Use when a metabolomics intensity matrix (compounds × runs) spans multiple acquisition batches, QC samples are interspersed in the run order, and signal drift/batch effects must be removed before differential abundance testing.

## When NOT to use

- The matrix contains no interspersed QC samples — QC-RLSC/QC-RSC are unusable and the QC-vs-biological fit checks cannot run
- Multivariate (cross-metabolite) outlier detection is required — the article notes treatment is currently feature-level, and multivariate methods like SERFF and TIGER were not benchmarked within Metanorm
- Data are already normalized or the goal is only statistical inference — applying batch correction would alter p-values without addressing a drift problem (note multiple testing correction was explicitly out of scope here)

## Inputs

- Unnormalized metabolomics intensity matrix (rows = compounds, columns = sample runs), e.g. BioHEART plasma data: 1359 runs × 53 targeted metabolites
- Batch vector assigning each sample run to an acquisition batch (15 batches in the BioHEART case)
- Sample-type vector labeling each run as 'QC' or another type
- Optional output directory for diagnostic per-compound pre/post-normalization plots

## Outputs

- Normalized intensity matrix (same compound × run dimensions)
- PCA score plots before and after normalization labeled by batch
- Per-compound pre- vs post-normalization intensity-vs-order drift plots written to plotdir
- Downstream DAM table per normalization method (Wilcoxon rank sum, p < 0.05) for cross-method overlap comparison

## How to apply

Log-transform the intensity matrix (base e) before normalization, then apply batchwise normalization per compound with the metanorm() function, passing the batch vector and the sample-type vector ('QC' vs other). Prefer model = 'tGAM' (REML-fitted thin plate regression splines via mgcv) with QCcheck = TRUE, so QC and biological samples are used jointly and discrepancies between them are flagged; rGAM and rLOESS (loess.as from fANCOVA with Tukey biweight downweighting) are faster alternatives. Classical QC-RLSC (loess, degree 2, span by LOOCV/GCV) and QC-RSC (smooth.spline) use QC samples only, so always inspect the per-compound pre- vs post-normalization drift plots and the pre/post PCA score plots (plotPCA) to check batch effects actually diminished. Judge success not by a larger DAM count — the article warns increased numbers of differentially abundant metabolites are a benchmarking pitfall (raw unnormalized BioHEART data yielded the highest DAM count) — but by improved replicate concordance, reduced drift, and agreement across robust methods (~92% joint DAM findings for tGAM/rGAM/rLOESS at p < 0.05 two-sided Wilcoxon tests).

## Related tools

- **Metanorm R package** (Primary implementation: metanorm() performs batchwise QC-based correction with model = tGAM/rGAM/rLOESS/QC-RLSC/QC-RSC; plotPCA() and the plotdir diagnostics verify performance) — https://github.com/UGent-LIMET/Metanorm
- **mgcv** (Fits the tGAM/rGAM normalization curves as (generalized) additive models with thin plate regression splines under REML)
- **fANCOVA** (Provides loess.as used for the cross-validated robust LOESS (rLOESS) fit with Tukey biweight downweighting)

## Examples

```
normdat <- metanorm(rawdata, model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/metanormOut/"); plotPCA(normdat, type = batch)
```

## Evaluation signals

- Output matrix preserves the input dimensions (compounds × runs, e.g. 53 × 1359 for BioHEART) and run/sample ordering
- PCA score plots colored by batch show visibly reduced batch separation after vs before normalization
- Per-compound pre/post drift plots inspected for a decent number of compounds — e.g. the README's QC-RLSC example where the PC plot looks clean but compound 4's drift in biological samples is problematic
- QCcheck = TRUE raises no unexplained discrepancies between QC and biological samples, or flagged ones receive analyst scrutiny
- Downstream sanity check: raw unnormalized data yields the highest DAM count (inflation signal), and tGAM/rGAM/rLOESS agree on ~92% of joint DAM findings while SERFF-type methods disagree (29–41%)

## Limitations

- QC-RLSC as prescribed uses QC samples only, which can misfit biological-sample drift (the README demonstrates a case where the PCA looks fine but compound-level drift in non-QC samples is wrong)
- Outlier treatment is feature-level only; multivariate outlier detection is not implemented and automated pre-normalization outlier removal is nontrivial and not provided
- An increased number of differentially abundant metabolites is NOT evidence of better normalization — raw data can show the highest DAM count
- Unexpected differences between QC and biological samples require analyst scrutiny rather than automated handling
- Requires R >= 4.4.0; SERFF and TIGER (multivariate methods) must be run separately (SERFF via its Shiny interface, TIGER via its R package) and were not benchmarked in Metanorm itself

## Evidence

- [readme] The package implements three (new) robust normalization methods (tGAM, rGAM and rLOESS), alongside formerly proposed ones (QC-RLSC, QC-RSC).: [evidence span withheld — anchor status: not_found]
- [readme] We further recommend using both QC as well as biological samples for normalization, and to have metanorm check for discrepancies between QC and biological samples.: [evidence span withheld — anchor status: external_doc]
