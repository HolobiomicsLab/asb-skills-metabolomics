---
name: chromatographic-signal-preprocessing
description: Use when raw GCxGC-MS chromatograms (e.g., NetCDF files folded into 2D-TIC
  matrices) contain a steady increasing baseline, noise, or retention-time shifts
  that must be removed prior to MPCA or supervised modeling.
when_to_use_negative:
- Input is already a preprocessed or feature-table representation (e.g. an unfolded
  matrix ready for mixOmics or an m_prcomp score matrix) — baseline correction operates
  on raw folded 2D-TIC chromatograms
- The analysis goal is visualization customization only (use the plotting_chromatograms
  vignette / colorRamps palettes instead of re-running signal preprocessing)
- Supervised multivariate modeling is the immediate goal and the data already passed
  through the two main RGCxGC workflow steps (preprocessing and multivariate analysis)
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RGCxGC
  role: R package providing the preprocessing functions (baseline_corr, read_chrom,
    batch_2DCOW/twod_cow, m_prcomp, unfold_chrom) applied to the 2D-TIC data
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: asymetric least squares
  role: Algorithm implemented by baseline_corr to fit and remove the steady increasing
    baseline (Eilers 2004)
- name: RNetCDF
  role: Reads the raw chromatogram from a NetCDF file which is then folded into the
    2D-TIC
- name: ptw
  role: Provides the base warping functions (with1 & with2) used for 2D-COW peak alignment
- name: R
  role: 'Runtime environment for the RGCxGC vignette workflow (vignette: Introduction
    to RGCxGC package)'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:3f5993939f43c775473ca5c97366023b9779c951e4c77c3b6af28448ee1a3082
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
  - ptw
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromatographic-signal-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw GCxGC-MS chromatograms in the RGCxGC R package: import a NetCDF chromatogram and fold it into a 2D-TIC matrix, then enhance the signal via asymmetric least squares baseline correction, Whittaker smoothing, and 2D-COW peak alignment before multivariate analysis.

## When to use

Use when raw GCxGC-MS chromatograms (e.g., NetCDF files folded into 2D-TIC matrices) contain a steady increasing baseline, noise, or retention-time shifts that must be removed prior to MPCA or supervised modeling.

## When NOT to use

- Input is already a preprocessed or feature-table representation (e.g. an unfolded matrix ready for mixOmics or an m_prcomp score matrix) — baseline correction operates on raw folded 2D-TIC chromatograms
- The analysis goal is visualization customization only (use the plotting_chromatograms vignette / colorRamps palettes instead of re-running signal preprocessing)
- Supervised multivariate modeling is the immediate goal and the data already passed through the two main RGCxGC workflow steps (preprocessing and multivariate analysis)

## Inputs

- Raw GCxGC-MS chromatogram imported from a NetCDF file (folded 2D-TIC matrix, e.g. the 08GB sample)
- chromatogram object in the RGCxGC package environment
- Parameters for baseline_corr (chromatogram, ...) and optional 2D-COW parameters (reference, segments, max_warp, add_ref)

## Outputs

- preproc_GCxGC object containing the baseline-corrected 2D-TIC matrix (e.g. MTBLS08_bc)
- 2D chromatogram plot showing the baseline removed and separation space filled with more intense signal
- Optionally further preprocessed chromatograms: smoothed and 2D-COW-aligned versions for downstream analysis

## How to apply

Load the raw imported chromatogram as a 2D-TIC matrix in the RGCxGC environment (import from a NetCDF file via read_chrom). Invoke baseline_corr on the chromatogram object: this fits and removes the steady increasing baseline using the asymmetric least squares (ALS) algorithm of Eilers 2004, returning a preproc_GCxGC object with the corrected 2D-TIC matrix. Complementary signal-enhancement steps in the same pipeline are Whittaker-smoother-based smoothing (Eilers 2003) and 2D-COW peak alignment via batch_2DCOW/twod_cow (coupled from the ptw package). Validate correction by inspecting the corrected 2D-TIC plot: the baseline ramp should be gone and the separation space filled with more intense signal (e.g., more intense blue in the filled.contour plot), as shown for the MTBLS08 chromatogram from dataset MTBLS579. After preprocessing, the chromatograms can be carried into multivariate analysis (m_prcomp) or unfolded with unfold_chrom for supervised mixOmics models.

## Related tools

- **RGCxGC** (R package providing the preprocessing functions (baseline_corr, read_chrom, batch_2DCOW/twod_cow, m_prcomp, unfold_chrom) applied to the 2D-TIC data) — https://github.com/DanielQuiroz97/RGCxGC
- **asymetric least squares** (Algorithm implemented by baseline_corr to fit and remove the steady increasing baseline (Eilers 2004))
- **RNetCDF** (Reads the raw chromatogram from a NetCDF file which is then folded into the 2D-TIC)
- **ptw** (Provides the base warping functions (with1 & with2) used for 2D-COW peak alignment)
- **R** (Runtime environment for the RGCxGC vignette workflow (vignette: Introduction to RGCxGC package))

## Examples

```
MTBLS08_bc <- baseline_corr(MTBLS08)
```

## Evaluation signals

- The steady increasing baseline is absent from the corrected 2D-TIC plot (baseline-correction section of the vignette)
- The separation space in the corrected chromatogram plot is filled with more intense blue, indicating enhanced signal relative to the raw chromatogram
- The returned object is a preproc_GCxGC object containing the corrected 2D-TIC matrix (e.g. MTBLS08_bc)
- The preprocessing chain matches the vignette's two main steps: preprocessing (smoothing, baseline correction, peak alignment) after data import
- Downstream m_prcomp runs successfully on the preprocessed chromatograms, yielding accessible score and loading matrices

## Limitations

- baseline_corr assumes a steady increasing baseline; the article does not describe handling of other baseline shapes or strongly curved backgrounds
- Peak alignment via 2D-COW requires specifying reference, segments, and max_warp parameters, which affect warping quality and are not auto-tuned in the surfaced workflow
- Preprocessing is a prerequisite for the package's multivariate analysis (MPCA); skipping it can leave noise that degrades downstream separation of sample groups
- No changelog was found for the repository, so function behavior may change between the paper and the current version

## Evidence
