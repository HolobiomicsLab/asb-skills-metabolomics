---
name: two-dimensional-signal-preprocessing
description: Use when raw GC×GC-MS data (NetCDF folded into a 2D-TIC) shows noise
  or baseline drift and must be smoothed, baseline-corrected, or aligned (e.g. wsmooth
  with penalty=1, lambda=1e1) prior to MPCA.
when_to_use_negative:
- The chromatograms are already preprocessed and the goal is downstream multivariate
  analysis (MPCA via m_prcomp) rather than signal enhancement
- The data is one-dimensional chromatography or non-chromatographic signal — the RGCxGC
  pipeline assumes a two-dimensional 2D-TIC from GC×GC-MS
- The input has not yet been imported from NetCDF into a 2D-TIC chromatogram object,
  since preprocessing functions operate on folded chromatogram objects
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RGCxGC
  role: R package providing read_chrom (NetCDF import/2D-TIC folding), baseline_corr
    (asymmetric least squares), wsmooth (Whittaker smoother), and batch_2DCOW/twod_cow
    (2D-COW alignment)
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Execution environment for the RGCxGC preprocessing workflow and for saving
    .rda chromatogram objects
- name: ptw
  role: Provides base functions (with1 & with2) coupled by RGCxGC for warping/alignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:15861d0b313e480255c6430167c004d4ab498e31b08091e946820eec4d2767a5
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
  - ptw
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# two-dimensional-signal-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing pipeline for GC×GC-MS 2D-TIC chromatograms implemented in the R RGCxGC package: signal enhancement via Whittaker smoothing, asymmetric-least-squares baseline correction, and 2D-COW peak alignment before multivariate analysis.

## When to use

Use when raw GC×GC-MS data (NetCDF folded into a 2D-TIC) shows noise or baseline drift and must be smoothed, baseline-corrected, or aligned (e.g. wsmooth with penalty=1, lambda=1e1) prior to MPCA.

## When NOT to use

- The chromatograms are already preprocessed and the goal is downstream multivariate analysis (MPCA via m_prcomp) rather than signal enhancement
- The data is one-dimensional chromatography or non-chromatographic signal — the RGCxGC pipeline assumes a two-dimensional 2D-TIC from GC×GC-MS
- The input has not yet been imported from NetCDF into a 2D-TIC chromatogram object, since preprocessing functions operate on folded chromatogram objects

## Inputs

- Raw GC×GC-MS chromatogram imported from a NetCDF file and folded into a 2D-TIC
- Baseline-corrected chromatogram object (e.g. 08GB.rda / MTBLS08_bc from dataset MTBLS579)
- For alignment: a reference chromatogram plus sample chromatograms with segment and max_warp settings

## Outputs

- Smoothed chromatogram object (e.g. MTBLS08_sm1.rda) smoothed along the first chromatographic dimension
- Baseline-corrected chromatogram
- 2D-COW-aligned chromatogram batch
- Unfolded 2D chromatogram matrices ready for mixOmics supervised models

## How to apply

Import the raw chromatogram from a NetCDF file and fold it into the two-dimensional Total Intensity Chromatogram (2D-TIC) with read_chrom (setting mod_time, sam_rate, x_cut/y_cut). Then apply the preprocessing chain: baseline correction via baseline_corr, which is based on asymmetric least squares, followed by Whittaker smoothing via wsmooth. For the smoothed variant of the baseline-corrected 08GB chromatogram (MTBLS08_bc), call wsmooth with penalty = 1 (linear penalty) and lambda = 1e1; smoothing is carried out along the first chromatographic dimension, i.e., on every chromatographic modulation, and the result is saved as MTBLS08_sm1.rda. For retention-time misalignment across samples, apply 2D-COW alignment (batch_2DCOW/twod_cow) with a reference chromatogram, sample set, segment definition, and max_warp. The preprocessed chromatograms then feed multivariate analysis (MPCA via m_prcomp) or are unfolded with unfold_chrom for supervised mixOmics models.

## Related tools

- **RGCxGC** (R package providing read_chrom (NetCDF import/2D-TIC folding), baseline_corr (asymmetric least squares), wsmooth (Whittaker smoother), and batch_2DCOW/twod_cow (2D-COW alignment)) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Execution environment for the RGCxGC preprocessing workflow and for saving .rda chromatogram objects)
- **ptw** (Provides base functions (with1 & with2) coupled by RGCxGC for warping/alignment)

## Examples

```
R> MTBLS08_sm1 <- wsmooth(MTBLS08_bc, penalty = 1, lambda = 1e1)
```

## Evaluation signals

- The smoothed variant MTBLS08_sm1 is reproduced by wsmooth on the baseline-corrected 08GB chromatogram with penalty = 1 and lambda = 1e1
- Smoothing is applied along the first chromatographic dimension, i.e., independently on every chromatographic modulation, not across the second dimension
- The baseline-corrected output shows reduced baseline drift consistent with asymmetric least squares correction before smoothing
- The resulting object persists as a valid .rda chromatogram (e.g. MTBLS08_sm1.rda) with the 2D-TIC dimensions preserved
- After preprocessing, the chromatograms can be fed to m_prcomp without dimension errors, indicating the three dimensions typical for 2D-TIC were handled correctly

## Limitations

- Preprocessing order matters: smoothing here is applied to the baseline-corrected chromatogram, so skipping baseline correction changes the smoothed result
- Parameter choice matters: the documented reproduction uses penalty = 1 and lambda = 1e1; other lambda/penalty settings yield different smoothed variants
- The vignette-based documentation leaves no changelog ('No changelog found.'), so package function behavior should be verified against the installed version

## Evidence

    ]"
