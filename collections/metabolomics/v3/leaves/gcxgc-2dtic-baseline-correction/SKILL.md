---
name: gcxgc-2dtic-baseline-correction
description: Use when a raw GCxGC chromatogram imported from a NetCDF file as a 2D-TIC
  matrix shows a steadily increasing baseline that must be removed before multivariate
  analysis or peak alignment.
when_to_use_negative:
- Input is not a chromatographic signal with a steady baseline drift (e.g., already
  a preprocessed or feature-level table).
- The chromatogram has not yet been imported/folded into a 2D-TIC matrix, and other
  preprocessing steps like smoothing or 2D-COW alignment are the actual target rather
  than baseline removal.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RGCxGC
  role: R package whose baseline_corr function implements asymmetric least squares
    baseline correction on 2D-TIC chromatograms
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: asymetric least squares
  role: Algorithm underlying the baseline_corr baseline correction
- name: R
  role: Language environment in which the RGCxGC package and baseline_corr are executed
- name: RNetCDF
  role: Supports importing the raw chromatogram from a NetCDF file into the 2D-TIC
    matrix prior to correction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:d11d13c246f25148b30c44234882faa2016b812bb2ed4852ddbdbcbd83c4b489
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
  - asymetric least squares
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gcxgc-2dtic-baseline-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes the steady increasing baseline from an imported GCxGC 2D-TIC chromatogram using the asymmetric least squares (ALS) algorithm via the baseline_corr function in the RGCxGC R package, yielding a corrected preproc_GCxGC object ready for downstream smoothing, alignment, or MPCA.

## When to use

Use when a raw GCxGC chromatogram imported from a NetCDF file as a 2D-TIC matrix shows a steadily increasing baseline that must be removed before multivariate analysis or peak alignment.

## When NOT to use

- Input is not a chromatographic signal with a steady baseline drift (e.g., already a preprocessed or feature-level table).
- The chromatogram has not yet been imported/folded into a 2D-TIC matrix, and other preprocessing steps like smoothing or 2D-COW alignment are the actual target rather than baseline removal.

## Inputs

- Raw GCxGC chromatogram imported from a NetCDF file as a 2D-TIC matrix (RGCxGC chromatogram object)

## Outputs

- Baseline-corrected chromatogram stored as a preproc_GCxGC object containing the corrected 2D-TIC matrix (e.g., MTBLS08_bc)

## How to apply

Load the raw imported chromatogram (e.g., the 08GB sample read from a NetCDF file and folded into a 2D-TIC matrix) within the RGCxGC package environment. Invoke baseline_corr on the chromatogram object; the function fits and subtracts the steady increasing baseline using the asymmetric least squares (ALS) algorithm of Eilers 2004. Store the result as a preproc_GCxGC object (e.g., MTBLS08_bc) containing the corrected 2D-TIC matrix. Validate the correction by plotting the corrected chromatogram: the baseline should be removed and the separation space filled with a more intense blue, consistent with the baseline-correction section of the vignette. The corrected object can then feed further preprocessing (Whittaker smoothing, 2D-COW alignment) or MPCA via m_prcomp.

## Related tools

- **RGCxGC** (R package whose baseline_corr function implements asymmetric least squares baseline correction on 2D-TIC chromatograms) — https://github.com/DanielQuiroz97/RGCxGC
- **asymetric least squares** (Algorithm underlying the baseline_corr baseline correction)
- **R** (Language environment in which the RGCxGC package and baseline_corr are executed)
- **RNetCDF** (Supports importing the raw chromatogram from a NetCDF file into the 2D-TIC matrix prior to correction)

## Examples

```
MTBLS08_bc <- baseline_corr(MTBLS08_chrom)
```

## Evaluation signals

- Plot of the corrected 2D-TIC shows the steady increasing baseline removed.
- The separation space in the plotted chromatogram is filled with a more intense blue after correction.
- Result is a preproc_GCxGC object (not a raw chromatogram object) containing a corrected 2D-TIC matrix of unchanged dimensions.
- Corrected chromatogram is consistent with the baseline-correction example in the RGCxGC vignette.

## Limitations

- Baseline correction targets a steady increasing baseline; other noise components require complementary preprocessing such as Whittaker smoothing or 2D-COW peak alignment.
- Correction operates on the 2D-TIC matrix after import from NetCDF; it cannot be applied directly to raw vendor or NetCDF files.

## Evidence

- [intro] ALS removes steady increasing baseline: [evidence span withheld — anchor status: external_doc]
    ]"
