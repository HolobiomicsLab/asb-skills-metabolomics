---
name: ppm-region-selection-for-shift-correction
description: Use when processing 1D 1H NMR metabolomics spectra (e.g. Bruker/Varian
  FID-derived) in NMRProcFlow and overlaid spectra show resonance shifts within specific
  ppm areas that baseline correction and ppm-scale calibration cannot fix.
when_to_use_negative:
- The spectra are raw, uncalibrated FIDs — ppm-scale calibration and baseline correction
  must be done first, since realignment assumes calibrated spectra
- The chemical shift differences are not uncontrolled drift between spectra but a
  uniform offset that a single ppm-scale calibration step can fix
- A region has different numbers of peaks across spectra — Parametric Time Warping
  there violates its assumption and will not produce a better result
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3382
tools:
- name: NMRProcFlow
  role: GUI tool hosting the interactive ppm-area selection and the realignment (Least-Square
    / Parametric Time Warping) step
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: NMR spectra processing application (R Shiny GUI) in which the realignment
    of ppm areas is executed
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: Client–server spectra viewer used to visually verify that resonances in the
    selected ppm areas are aligned across spectra
  repo: https://github.com/INRA/NMRProcFlow
- name: Rcpp
  role: Package through which the C++ implementations of the processing/alignment
    algorithms are called from R
  repo: https://github.com/INRA/NMRProcFlow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:0eacf4f8b016c11af3919f818dae2f1b27933407fda871ec288f682a6fb730a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRspec
  - NMRviewer
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ppm-region-selection-for-shift-correction

## Summary

Interactive selection of ppm areas in 1D 1H NMR spectra whose resonances show uncontrolled chemical shift variations between samples, followed by realignment of each selected area using NMRProcFlow's Least-Square algorithm or Parametric Time Warping. It corrects peak drift prior to binning so that metabolite signals line up across the spectra matrix.

## When to use

Use when processing 1D 1H NMR metabolomics spectra (e.g. Bruker/Varian FID-derived) in NMRProcFlow and overlaid spectra show resonance shifts within specific ppm areas that baseline correction and ppm-scale calibration cannot fix.

## When NOT to use

- The spectra are raw, uncalibrated FIDs — ppm-scale calibration and baseline correction must be done first, since realignment assumes calibrated spectra
- The chemical shift differences are not uncontrolled drift between spectra but a uniform offset that a single ppm-scale calibration step can fix
- A region has different numbers of peaks across spectra — Parametric Time Warping there violates its assumption and will not produce a better result

## Inputs

- Set of calibrated, baseline-corrected 1D 1H NMR spectra (e.g. from Bruker or Agilent/Varian FID, or nmrML) loaded into NMRProcFlow
- User-selected ppm areas exhibiting uncontrolled chemical shift variations between spectra

## Outputs

- Realigned spectra within each selected ppm area, viewable in the NMR spectra viewer
- Realigned spectra feeding downstream binning (Intelligent, variable size or fixed width) into a spectra data matrix

## How to apply

Load the set of calibrated 1D 1H NMR spectra into the NMRspec application of NMRProcFlow (implemented mainly in R, with C++ processing algorithms exposed via Rcpp) and inspect them overlaid or stacked in the NMR spectra viewer. Interactively zoom on regions and select the set of ppm areas whose resonances show uncontrolled chemical shift variations between spectra — this visual, user-driven region selection is the point of the skill, since automated batch-mode alignment still requires user expertise. For each selected ppm area, apply the Least-Square realignment algorithm; where peaks are intertwined and the Least-Square adaptation does not fit, switch to Parametric Time Warping (Bloemberg et al. 2010), which gives better results provided all spectra have the same number of peaks in that region. Realignment of a set of ppm areas is performed before binning (Intelligent, variable size or fixed width), so mis-selected or mis-aligned regions propagate artifacts into the final data matrix.

## Related tools

- **NMRProcFlow** (GUI tool hosting the interactive ppm-area selection and the realignment (Least-Square / Parametric Time Warping) step) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (NMR spectra processing application (R Shiny GUI) in which the realignment of ppm areas is executed) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Client–server spectra viewer used to visually verify that resonances in the selected ppm areas are aligned across spectra) — https://github.com/INRA/NMRProcFlow
- **Rcpp** (Package through which the C++ implementations of the processing/alignment algorithms are called from R) — https://github.com/INRA/NMRProcFlow

## Examples

```
docker pull docker.io/nmrprocflow/npflow; sh ./npflow start
```

## Evaluation signals

- Re-inspection of the overlaid/stacked spectra in the NMR spectra viewer shows resonances within each selected ppm area aligned across all spectra
- The spectra data matrix exported after realignment has one row per input spectrum (no spectra dropped or duplicated)
- For regions realigned with Parametric Time Warping, every spectrum contains the same number of peaks in that ppm area (the method's stated precondition)
- Aligned regions feed binning without introducing new peak-shift artifacts that Adaptive Intelligent Binning would otherwise have to absorb

## Limitations

- Spectra alignment remains a challenge requiring user expertise; the interactive region-selection approach is chosen because fully automatic batch-mode workflows usable by non-experts are not yet mature
- Parametric Time Warping only performs well when all spectra have the same number of peaks in the realigned region
- For crowded regions with heavy peak overlap where neither method suffices, NMRProcFlow plans a future local signal deconvolution-based quantification method, not yet available

## Evidence

- [intro] Re-alignment scope: "re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [methods] ppm-area realignment as processing step: "the realignment of a set of ppm areas"
- [abstract] PTW for intertwined peaks: "1D spectra processing often requires an expert eye for disentangling the intertwined peaks."
- [readme] Full processing pipeline context: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
- [intro] Batch-mode limitation motivating interactive selection: "the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode (regarded as a black-box)"
