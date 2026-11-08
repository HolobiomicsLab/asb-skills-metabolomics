---
name: whittaker-smoother-parameterization
description: Use when a GCxGC-MS chromatogram (e.g., a baseline-corrected 2D-TIC from
  the MTBLS579 dataset) still contains noise after baseline correction and signal
  smoothing along the first chromatographic dimension is required before multivariate
  analysis.
when_to_use_negative:
- The chromatogram has not yet been baseline-corrected — the RGCxGC pipeline applies
  baseline correction (asymmetric least squares) before smoothing variants like MTBLS08_sm1
  are generated.
- The goal is peak misalignment correction rather than noise reduction — use the 2D-COW
  alignment (batch_2DCOW / twod_cow) skill instead of smoothing.
- The input is 1D chromatography data or a feature table rather than a folded two-dimensional
  GCxGC 2D-TIC chromatogram.
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_3954
tools:
- name: RGCxGC
  role: Provides the wsmooth function that applies the Whittaker smoother to a 2D-TIC
    chromatogram along the first chromatographic dimension with a chosen penalty order
    and lambda
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: R
  role: Execution environment in which the RGCxGC package is loaded and the smoothed
    chromatogram object (MTBLS08_sm1.rda) is saved
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:ceabf103a04426a41070d093f4e9366c83ffba7555a955f40ee6a08ec0373aa1
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
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_3954
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# whittaker-smoother-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the Whittaker smoother (Eilers' penalized least-squares smoother) to a 2D-TIC GCxGC chromatogram via the RGCxGC wsmooth function, with an explicit penalty order and lambda. The linear-penalty variant (penalty=1, lambda=1e1) applied to the baseline-corrected 08GB chromatogram reproduces the smoothed variant MTBLS08_sm1.

## When to use

Use when a GCxGC-MS chromatogram (e.g., a baseline-corrected 2D-TIC from the MTBLS579 dataset) still contains noise after baseline correction and signal smoothing along the first chromatographic dimension is required before multivariate analysis.

## When NOT to use

- The chromatogram has not yet been baseline-corrected — the RGCxGC pipeline applies baseline correction (asymmetric least squares) before smoothing variants like MTBLS08_sm1 are generated.
- The goal is peak misalignment correction rather than noise reduction — use the 2D-COW alignment (batch_2DCOW / twod_cow) skill instead of smoothing.
- The input is 1D chromatography data or a feature table rather than a folded two-dimensional GCxGC 2D-TIC chromatogram.

## Inputs

- Baseline-corrected 2D-TIC GCxGC chromatogram object in R (e.g., 08GB.rda / MTBLS08_bc from the RGCxGC package environment)
- GCxGC-MS chromatogram originally imported from a NetCDF file and folded into a two-dimensional Total Intensity Chromatogram

## Outputs

- Smoothed 2D-TIC chromatogram object (e.g., MTBLS08_sm1.rda) with Whittaker smoothing applied along the first chromatographic dimension of every modulation

## How to apply

Load the baseline-corrected chromatogram object (e.g., 08GB.rda) into R from the RGCxGC package environment, then invoke wsmooth with penalty = 1 (linear penalty) and lambda = 1e1. The smoother is carried out along the first chromatographic dimension, i.e., independently on every chromatographic modulation of the 2D-TIC, since a linear penalty regularizes within-modulation retention-time traces rather than across the second dimension. The rationale is noise reduction prior to statistical analysis — the RGCxGC pipeline places smoothing alongside baseline correction and peak alignment as preprocessing before MPCA. With these parameters on MTBLS08_bc, the output is the smoothed chromatogram MTBLS08_sm1, which should be saved as MTBLS08_sm1.rda. Higher penalty orders or lambda values would correspond to different smoothed variants, so record the exact penalty/lambda pair used.

## Related tools

- **RGCxGC** (Provides the wsmooth function that applies the Whittaker smoother to a 2D-TIC chromatogram along the first chromatographic dimension with a chosen penalty order and lambda) — https://github.com/DanielQuiroz97/RGCxGC
- **R** (Execution environment in which the RGCxGC package is loaded and the smoothed chromatogram object (MTBLS08_sm1.rda) is saved)

## Examples

```
MTBLS08_sm1 <- wsmooth(MTBLS08_bc, penalty = 1, lambda = 1e1); save(MTBLS08_sm1, file = "MTBLS08_sm1.rda")
```

## Evaluation signals

- Running wsmooth on MTBLS08_bc with penalty = 1 and lambda = 1e1 yields a chromatogram identical to the documented smoothed variant MTBLS08_sm1.
- Smoothing is applied along the first chromatographic dimension only — each modulation's trace is smoothed independently, with no cross-modulation regularization.
- The resulting smoothed object can be saved and reloaded as MTBLS08_sm1.rda without dimension or metadata changes.
- The smoothed chromatogram retains the 2D-TIC structure (two retention-time dimensions) required for downstream MPCA via m_prcomp.
- The penalty order and lambda parameters of the run match the documented linear-penalty parameterization (penalty = 1, lambda = 1e1).

## Limitations

- The smoothing is defined per modulation along the first chromatographic dimension; it does not jointly smooth across the second dimension, so residual noise along the modulation axis is not addressed.
- Different penalty orders or lambda values produce different smoothed variants — the linear-penalty, lambda=1e1 setting specifically reproduces MTBLS08_sm1 and is not a universal default.
- RGCxGC offers common pre-processing algorithms including Whittaker smoothing, but the package documentation provides no changelog, so version-specific behavior may be undocumented.

## Evidence

- [readme] Whittaker smoother basis in README: [evidence span withheld — anchor status: not_found]
- [readme] Eilers 2003 reference for the smoother: [evidence span withheld — anchor status: external_doc]
