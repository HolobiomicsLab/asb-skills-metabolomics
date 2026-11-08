---
name: asymmetric-least-squares-baseline-estimation
description: Use when a raw GCxGC chromatogram (e.g. a 2D-TIC folded from a NetCDF
  file within the RGCxGC environment) shows a steady increasing baseline that must
  be removed before smoothing, peak alignment, or multivariate analysis.
when_to_use_negative:
- Input is already a preprocessed chromatogram (preproc_GCxGC object) or an unfolded
  feature table for supervised mixOmics models — re-correcting can distort intensities
- The goal is smoothing or peak alignment on an already baseline-corrected chromatogram
  — use the Whittaker smoother or twod_cow/batch_2DCOW instead
- The chromatogram has no steady increasing baseline (e.g. flat 2D-TIC background)
  — ALS fitting adds unnecessary signal modification
tools:
- name: RGCxGC
  role: Provides the baseline_corr function that implements the asymmetric least squares
    baseline correction on 2D-TIC chromatogram objects
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: asymetric least squares
  role: Algorithm fitted by baseline_corr to estimate and remove the steady increasing
    baseline (Eilers 2004)
- name: R
  role: Runtime environment for loading the 2D-TIC matrix and storing the corrected
    preproc_GCxGC object
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:f853ba52e5af72175d53b9fc71a39c52211cae9c1afdee23eb6c4ea737e56a84
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
  - asymetric least squares
  - ptw
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# asymmetric-least-squares-baseline-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Baseline correction of GCxGC 2D-TIC chromatograms via the asymmetric least squares (ALS) algorithm of Eilers 2004, implemented as the baseline_corr function in the RGCxGC R package. It removes the steady increasing baseline from raw 2D-TIC signals so downstream smoothing, peak alignment, and MPCA operate on corrected intensities.

## When to use

Use when a raw GCxGC chromatogram (e.g. a 2D-TIC folded from a NetCDF file within the RGCxGC environment) shows a steady increasing baseline that must be removed before smoothing, peak alignment, or multivariate analysis.

## When NOT to use

- Input is already a preprocessed chromatogram (preproc_GCxGC object) or an unfolded feature table for supervised mixOmics models — re-correcting can distort intensities
- The goal is smoothing or peak alignment on an already baseline-corrected chromatogram — use the Whittaker smoother or twod_cow/batch_2DCOW instead
- The chromatogram has no steady increasing baseline (e.g. flat 2D-TIC background) — ALS fitting adds unnecessary signal modification

## Inputs

- Raw 2D-TIC chromatogram object in R (RGCxGC chromatogram folded from an imported NetCDF file)
- R session with the RGCxGC package installed (CRAN or devtools::install_github("DanielQuiroz97/RGCxGC"))

## Outputs

- Baseline-corrected chromatogram stored as a preproc_GCxGC object (e.g. MTBLS08_bc) containing the corrected 2D-TIC matrix

## How to apply

Load the raw imported chromatogram (e.g. the 08GB sample from MTBLS579) as a 2D-TIC matrix in the RGCxGC package environment after NetCDF import. Invoke baseline_corr on the chromatogram object; the function fits the steady increasing baseline with the asymmetric least squares (ALS) algorithm of Eilers 2004 and subtracts it from the 2D-TIC. Store the returned object, which is a preproc_GCxGC object containing the corrected 2D-TIC matrix (e.g. MTBLS08_bc). Validate by inspecting or plotting the corrected matrix: the baseline removed should leave the separation space filled with a more intense blue in the chromatogram plot. Baseline correction is the baseline-removal step of the RGCxGC preprocessing pipeline, alongside Whittaker smoothing and 2D-COW peak alignment.

## Related tools

- **RGCxGC** (Provides the baseline_corr function that implements the asymmetric least squares baseline correction on 2D-TIC chromatogram objects) — https://github.com/DanielQuiroz97/RGCxGC
- **asymetric least squares** (Algorithm fitted by baseline_corr to estimate and remove the steady increasing baseline (Eilers 2004))
- **R** (Runtime environment for loading the 2D-TIC matrix and storing the corrected preproc_GCxGC object)

## Examples

```
MTBLS08_bc <- baseline_corr(MTBLS08)
```

## Evaluation signals

- Result is a preproc_GCxGC object whose corrected 2D-TIC matrix has the same dimensionality as the input raw chromatogram
- Plot of the corrected chromatogram (filled.contour-based plot) shows the steady increasing baseline removed and the separation space filled with a more intense blue
- Correction is consistent with the baseline-correction section of the RGCxGC vignette when reproduced on the 08GB sample from MTBLS579
- Corrected 2D-TIC retains the chromatographic peaks while the additive increasing background is subtracted (no reversal or loss of peak structure)

## Limitations

- Targets a steady increasing baseline; other baseline shapes or noise components must be handled by the companion smoothing (Whittaker) or peak-alignment (2D-COW) preprocessing steps
- Applied within the RGCxGC package environment on 2D-TIC matrices; not directly applicable to point-detection peak tables or non-GCxGC data

## Evidence

- [readme] ALS algorithm implemented by baseline_corr: "baseline correction based on [asymetric least
squares](https://pubs.acs.org/doi/abs/10.1021/ac034800e)"
- [intro] Baseline correction is a preprocessing step to reduce noise before statistical analysis: "noise can be significantly reduced by using preprocessing algorithms, herein smoothing, baseline correction, and peak alignment"
- [intro] Input is a 2D-TIC chromatogram folded from NetCDF: "the raw chromatogram is importing from a **NetCDF** file and is folded into the two-dimensional Total Intensity Chromatogram (2D-TIC)"
- [abstract] baseline_corr operates on a chromatogram object: ""baseline_corr": [
      "chromatogram",
      "..."
    ]"
