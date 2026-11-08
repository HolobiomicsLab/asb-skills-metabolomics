---
name: nmr-spectral-realignment
description: Use when overlaid/stacked 1D 1H NMR spectra show uncontrolled chemical
  shift variations between spectra after ppm-scale calibration, i.e. peaks of the
  same metabolite drift across the spectral dimension.
when_to_use_negative:
- The goal is ppm-scale calibration, baseline correction (locally or fully), or binning
  (Intelligent, variable size or fixed width) rather than correcting chemical shift
  variations between spectra — these are separate NMRProcFlow processing stages.
- The spectral region with intertwined peaks does not contain the same amount of peaks
  across all spectra, which violates the stated precondition for Parametric Time Warping.
- Input spectra have not yet been chemically shift calibrated; realignment is meant
  for uncontrolled variations remaining after calibration.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3665
tools:
- name: NMRProcFlow
  role: GUI application providing the realignment of a set of ppm areas, with the
    NMR spectra viewer as the central interactive tool for selecting ppm areas
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: NMR spectra processing application (R-based, R Shiny GUI) into which the calibrated
    1D 1H spectra are loaded and where realignment is executed
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Open source language in which NMRspec and the realignment processing are implemented
  repo: https://github.com/INRA/NMRProcFlow
- name: Rcpp
  role: Package used to implement processing algorithms (including realignment) in
    C++ callable from R
  repo: https://github.com/INRA/NMRProcFlow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:a44e44a3636c2d3a9888e2fb5c9f6300f33da1d68fcf7450d1560f64be6efca8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRspec
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3665
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nmr-spectral-realignment

## Summary

Re-align resonances showing uncontrolled chemical shift variations across a set of calibrated 1D 1H NMR spectra, using NMRProcFlow's Least-Square realignment on selected ppm areas and Parametric Time Warping for regions with intertwined peaks.

## When to use

Use when overlaid/stacked 1D 1H NMR spectra show uncontrolled chemical shift variations between spectra after ppm-scale calibration, i.e. peaks of the same metabolite drift across the spectral dimension.

## When NOT to use

- The goal is ppm-scale calibration, baseline correction (locally or fully), or binning (Intelligent, variable size or fixed width) rather than correcting chemical shift variations between spectra — these are separate NMRProcFlow processing stages.
- The spectral region with intertwined peaks does not contain the same amount of peaks across all spectra, which violates the stated precondition for Parametric Time Warping.
- Input spectra have not yet been chemically shift calibrated; realignment is meant for uncontrolled variations remaining after calibration.

## Inputs

- Set of calibrated 1D 1H NMR spectra loaded into NMRProcFlow/NMRspec (FID from Bruker or Agilent/Varian, or nmrML format)
- User-selected ppm areas whose resonances show uncontrolled chemical shift variations between spectra

## Outputs

- Realigned spectra exported as a data matrix (e.g. aligned_spectra.csv)
- Realigned spectra viewable/re-inspectable in the NMR spectra viewer

## How to apply

Load the set of calibrated 1D 1H NMR spectra into the NMRspec application of NMRProcFlow (R-based, with C++ processing algorithms exposed through the Rcpp package) and launch the NMR spectra viewer. Visually inspect the overlaid or stacked spectra and select the set of ppm areas whose resonances show uncontrolled chemical shift variations between spectra. For each selected ppm area, apply the Least-Square realignment algorithm to correct the chemical shift offsets; this is the default route for regions with well-resolved peaks. For regions with intertwined peaks where Least-Square adaptation does not fit, apply Parametric Time Warping (Bloemberg et al. 2010) instead, which produces a better result provided that all spectra have the same amount of peaks in that region. Export the realigned spectra as a data matrix (e.g. aligned_spectra.csv) and re-inspect the selected ppm areas in the viewer to confirm alignment across spectra.

## Related tools

- **NMRProcFlow** (GUI application providing the realignment of a set of ppm areas, with the NMR spectra viewer as the central interactive tool for selecting ppm areas) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (NMR spectra processing application (R-based, R Shiny GUI) into which the calibrated 1D 1H spectra are loaded and where realignment is executed) — https://github.com/INRA/NMRProcFlow
- **R** (Open source language in which NMRspec and the realignment processing are implemented) — https://github.com/INRA/NMRProcFlow
- **Rcpp** (Package used to implement processing algorithms (including realignment) in C++ callable from R) — https://github.com/INRA/NMRProcFlow

## Examples

```
sh ./npflow start  # launch NMRProcFlow (after: docker pull docker.io/nmrprocflow/npflow), then apply Least-Square/PTW realignment interactively in the NMR spectra viewer
```

## Evaluation signals

- Exported data matrix exists and contains the same number of spectral rows as the input set of spectra.
- On re-inspection in the NMR spectra viewer, resonances within the selected ppm areas are aligned across spectra.
- Least-Square was applied to each selected ppm area; Parametric Time Warping was only used on intertwined-peak regions where all spectra have the same amount of peaks.
- NMRProcFlow offers at least two methods per spectra processing stage (Supplementary Information S1), so the chosen realignment method can be confirmed against that menu.

## Limitations

- Spectra alignment requires user expertise and remains one of the unsolved challenges in 1D NMR spectra processing for high-throughput metabolomics (Larive et al. 2015).
- Parametric Time Warping produces a better result only provided that all spectra have the same amount of peaks in the targeted region; it fails where peak counts differ.
- Realignment is an interactive, expert-driven step rather than a fully automated batch-mode operation; NMRProcFlow's authors note batch-mode black-box processing by non-experts has not reached full maturity.
- Crowded regions with heavy peak overlap may need future local signal deconvolution for quantification, which was only planned in a future version of NMRProcFlow.

## Evidence

- [intro] re-alignment of resonances having uncontrolled variations in chemical shifts between spectra: "removal of resonance signals of solvents and other contaminants, re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [methods] the realignment of a set of ppm areas: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or fully), the realignment of a set of ppm areas"
- [methods] some processing algorithms have been implemented in C + + using the Rcpp package: "some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011)"
- [abstract] covers all spectra processing steps including baseline correction, chemical shift calibration: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
- [readme] the NMR spectra viewer is the central tool of NMRProcFlow: "The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application."
