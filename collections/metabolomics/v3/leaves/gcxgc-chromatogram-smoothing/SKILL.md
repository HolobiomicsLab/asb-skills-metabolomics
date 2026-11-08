---
name: gcxgc-chromatogram-smoothing
description: Use when a 2D-TIC GC×GC chromatogram (e.g., imported from a NetCDF file
  and baseline-corrected with baseline_corr) still contains noise and must be denoised
  prior to 2D-COW peak alignment or multivariate analysis.
when_to_use_negative:
- The chromatogram has not yet been baseline-corrected — the RGCxGC pipeline applies
  smoothing to the output of baseline_corr, not to the raw 2D-TIC imported from NetCDF.
- The immediate goal is peak alignment rather than noise reduction — smoothing is
  a preprocessing step; misalignment is addressed by 2D-COW (batch_2DCOW / twod_cow),
  not by wsmooth.
- The analysis goal is supervised modeling or dimensionality reduction — for those,
  RGCxGC exports chromatograms via unfold_chrom for mixOmics or applies MPCA via m_prcomp
  instead.
tools:
- name: RGCxGC
  role: Provides the wsmooth function implementing the Whittaker smoother on two-dimensional
    GC×GC chromatograms
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Runtime environment in which the RGCxGC vignette workflow (load .rda, run
    wsmooth, save smoothed object) is executed
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:c52f621d1b6e885a36635a8cef61632ae0ef0f06fe2afc22ce8643f310f9356c
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
  - RNetCDF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gcxgc-chromatogram-smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Whittaker smoothing to a baseline-corrected GC×GC chromatogram via the RGCxGC wsmooth function, running along the first chromatographic dimension (every modulation) with a chosen penalty order (1 or 2) and lambda (e.g., 1e1) to reduce noise before peak alignment.

## When to use

Use when a 2D-TIC GC×GC chromatogram (e.g., imported from a NetCDF file and baseline-corrected with baseline_corr) still contains noise and must be denoised prior to 2D-COW peak alignment or multivariate analysis.

## When NOT to use

- The chromatogram has not yet been baseline-corrected — the RGCxGC pipeline applies smoothing to the output of baseline_corr, not to the raw 2D-TIC imported from NetCDF.
- The immediate goal is peak alignment rather than noise reduction — smoothing is a preprocessing step; misalignment is addressed by 2D-COW (batch_2DCOW / twod_cow), not by wsmooth.
- The analysis goal is supervised modeling or dimensionality reduction — for those, RGCxGC exports chromatograms via unfold_chrom for mixOmics or applies MPCA via m_prcomp instead.

## Inputs

- Baseline-corrected GC×GC chromatogram object (e.g., MTBLS08_bc / 08GB.rda) from the RGCxGC package
- Penalty order parameter (1 for linear, 2 for quadratic)
- Smoothing parameter lambda (vignette value: 1e1)

## Outputs

- Smoothed chromatogram object (e.g., MTBLS08_sm1 or MTBLS08_sm2), smoothed along the first chromatographic dimension on every modulation
- Saved .rda file of the smoothed chromatogram (e.g., MTBLS08_sm2.rda) for downstream peak alignment

## How to apply

Load the baseline-corrected chromatogram object (e.g., 08GB.rda / MTBLS08_bc, produced by baseline_corr on the 2D-TIC folded from the NetCDF file) into R with the RGCxGC package. Invoke wsmooth on that chromatogram, choosing the penalty order — penalty = 1 (linear penalty) yields the first smoothed variant (MTBLS08_sm1), penalty = 2 (quadratic penalty) yields the second smoothed variant (MTBLS08_sm2) — with lambda set to 1e1 (10) as in the vignette. The Whittaker smoother (Eilers 2003, 'A Perfect Smoother') is applied along the first chromatographic dimension, i.e., independently on every chromatographic modulation. Save the resulting smoothed chromatogram object to disk (e.g., MTBLS08_sm2.rda); the quadratic-penalty variant is the object later used as the peak-alignment input in the RGCxGC workflow.

## Related tools

- **RGCxGC** (Provides the wsmooth function implementing the Whittaker smoother on two-dimensional GC×GC chromatograms) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Runtime environment in which the RGCxGC vignette workflow (load .rda, run wsmooth, save smoothed object) is executed)

## Examples

```
MTBLS08_sm2 <- wsmooth(MTBLS08_bc, penalty = 2, lambda = 1e1)
```

## Evaluation signals

- The smoothed chromatogram object (e.g., MTBLS08_sm2) exists and was saved as MTBLS08_sm2.rda.
- wsmooth was called with the documented settings — penalty = 2 and lambda = 1e1 for the second variant; penalty = 1 and lambda = 1e1 for the first variant.
- Smoothing was performed along the first chromatographic dimension, i.e., on every chromatographic modulation, not across modulations.
- The input to wsmooth was the baseline-corrected chromatogram (MTBLS08_bc), matching the vignette ordering of preprocessing steps (smoothing, baseline correction, peak alignment).
- The smoothed variant intended for alignment (MTBLS08_sm2, quadratic penalty) is the object passed forward to the peak-alignment step.

## Limitations

- The vignette's exact parameterization is penalty ∈ {1, 2} with lambda = 1e1; other lambda or penalty choices would produce different smoothed variants not covered by these source cards.
- The smoother acts along the first chromatographic dimension only (per modulation); smoothing behavior along the second dimension is not described here.
- Smoothing is one of three preprocessing algorithms (smoothing, baseline correction, peak alignment) whose order matters: the smoothed MTBLS08_sm2 is generated from the baseline-corrected object and feeds the 2D-COW alignment step.

## Evidence
