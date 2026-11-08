---
name: chemical-shift-calibration-and-alignment
description: Use when 1D 1H NMR metabolomics spectra show uncontrolled chemical-shift
  variations between spectra that must be calibrated and realigned before binning
  or statistical analysis.
when_to_use_negative:
- The region of interest contains intertwined peaks but the spectra do not have the
  same amount of peaks there — Parametric Time Warping only gives a better result
  when peak counts match.
- Input is already a binned/bucketed feature table (Intelligent, variable size or
  fixed width binning) — realignment operates on calibrated spectra before binning.
- A fully automatic, black-box batch pipeline for non-expert use is required — the
  article states automated batch-mode workflows have not reached full maturity and
  interactive processing with a spectra viewer is still needed.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0593
tools:
- name: NMRProcFlow
  role: GUI application used to calibrate the ppm scale and interactively realign
    a set of ppm areas of 1D NMR spectra
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: NMR spectra processing application (R/Shiny) in which the calibration and
    realignment of spectra are executed
- name: NMRviewer
  role: Client–server NMR spectra viewer used to visually explore spectra overlaid
    or stacked and to verify alignment across spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:062fe96c5fdb3006ff0a15689f86dd3d97a3e261387103751c3fd2c91c8858b7
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
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chemical-shift-calibration-and-alignment

## Summary

Interactive calibration of the ppm scale and realignment of resonances across sets of 1D 1H NMR spectra in NMRProcFlow, using the Least-Square realignment algorithm per selected ppm area or Parametric Time Warping (Bloemberg et al. 2010) for regions with intertwined peaks, so that uncontrolled chemical-shift variations between spectra do not distort downstream binning.

## When to use

Use when 1D 1H NMR metabolomics spectra show uncontrolled chemical-shift variations between spectra that must be calibrated and realigned before binning or statistical analysis.

## When NOT to use

- The region of interest contains intertwined peaks but the spectra do not have the same amount of peaks there — Parametric Time Warping only gives a better result when peak counts match.
- Input is already a binned/bucketed feature table (Intelligent, variable size or fixed width binning) — realignment operates on calibrated spectra before binning.
- A fully automatic, black-box batch pipeline for non-expert use is required — the article states automated batch-mode workflows have not reached full maturity and interactive processing with a spectra viewer is still needed.

## Inputs

- Set of calibrated 1D 1H (or 13C) NMR spectra / FID from Bruker or Agilent (Varian), including the Bruker folder structure with 1r files
- Spectra in nmrML format (also supported by NMRProcFlow)
- Interactively selected ppm areas whose resonances show chemical-shift variations between spectra

## Outputs

- Realigned spectra exported as a data matrix (e.g., aligned_spectra.csv) with the same number of spectral rows as the input set
- Overlaid/stacked spectra view in the NMR spectra viewer showing resonances aligned across spectra within the selected ppm areas

## How to apply

Load the set of calibrated 1D 1H (or 13C) NMR spectra — FID from Bruker or Agilent (Varian), including the Bruker folder structure with 1r files, or nmrML — into the NMRspec application of NMRProcFlow (implemented in R with a Shiny GUI, C++ algorithms via Rcpp, deployable with the npflow Docker image). In the NMR spectra viewer, inspect the spectra overlaid or stacked and interactively select the ppm areas whose resonances show uncontrolled chemical-shift variations between spectra. For each selected ppm area, apply the Least-Square realignment algorithm to correct the shifts between spectra. When a region contains intertwined peaks where the Least-Square adaptation does not fit, apply Parametric Time Warping (Bloemberg et al. 2010) instead, which produces a better result provided that all spectra have the same amount of peaks in that region. Export the realigned spectra as a data matrix and validate that it has the same number of spectral rows as the input set and that re-inspection in the NMR spectra viewer shows resonances aligned within the selected ppm areas — the article argues this interactive, expert-driven route is preferable to automatic batch-mode processing regarded as a black box.

## Related tools

- **NMRProcFlow** (GUI application used to calibrate the ppm scale and interactively realign a set of ppm areas of 1D NMR spectra) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (NMR spectra processing application (R/Shiny) in which the calibration and realignment of spectra are executed)
- **NMRviewer** (Client–server NMR spectra viewer used to visually explore spectra overlaid or stacked and to verify alignment across spectra)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- The exported realigned data matrix contains the same number of spectral rows as the input spectrum set.
- Re-inspection of the overlaid/stacked spectra in the NMR spectra viewer shows resonances within the selected ppm areas aligned across spectra.
- Each selected ppm area was processed with a method matched to its peak structure: Least-Square for simple shifts, Parametric Time Warping only where all spectra have the same amount of peaks.
- Calibration of the ppm scale was applied before realignment, so residual shifts observed in the viewer reflect alignment quality rather than calibration offsets.
- Downstream binning (Intelligent, variable size or fixed width) produces no split or duplicated buckets attributable to residual peak drift.

## Limitations

- Parametric Time Warping produces a better result only provided that all spectra have the same amount of peaks in the region; unequal peak counts break the method.
- The Least-Square realignment adaptation does not fit spectral regions containing intertwined peaks.
- Spectra alignment remains an open challenge in 1D NMR processing that requires user expertise; the interactive ppm-area selection cannot be replaced by a mature automatic batch-mode black box.

## Evidence

- [abstract] covers all spectra processing steps including baseline correction, chemical shift calibration and alignment.: "covers all spectra processing steps including baseline correction, chemical shift calibration and alignment."
- [intro] re-alignment of resonances having uncontrolled variations in chemical shifts between spectra: "re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [methods] NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R: "NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R"
- [readme] the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or fully), the realignment of a set of ppm areas: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or fully), the realignment of a set of ppm areas"
- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level."
