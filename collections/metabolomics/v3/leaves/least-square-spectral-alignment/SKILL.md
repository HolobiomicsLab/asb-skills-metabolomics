---
name: least-square-spectral-alignment
description: Use when overlaid 1D 1H NMR spectra show uncontrolled chemical shift
  variations between spectra in specific ppm areas, and peaks must be re-aligned before
  binning or statistical analysis of metabolomics data.
when_to_use_negative:
- Input spectra have not yet undergone ppm scale calibration or baseline correction
  — realignment is one step within the NMRProcFlow processing sequence that includes
  calibration of the ppm scale and base line correction
- The spectral region contains intertwined peaks and the spectra do not have the same
  amount of peaks in that region — Parametric Time Warping will not produce a good
  result in this case
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRProcFlow
  role: GUI application in which ppm areas are selected interactively and the Least-Square
    realignment is applied to spectra
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: The NMR spectra processing application of NMRProcFlow, implemented mainly
    in R, that performs the realignment of a set of ppm areas
- name: R
  role: Language in which the NMRspec spectra processing application is implemented
- name: Rcpp
  role: Package used to implement some processing algorithms in C++ within the Rnmr1D
    module
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:7c3bb56afcb4573fdb15a36ffc436ced68cc819318295d8fe02e8724ca52bed4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRspec
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# least-square-spectral-alignment

## Summary

Realignment of resonances showing uncontrolled chemical shift variations across a set of 1D 1H NMR spectra, applied interactively to selected ppm areas in NMRProcFlow using the Least-Square algorithm, with Parametric Time Warping as the fallback for regions of intertwined peaks.

## When to use

Use when overlaid 1D 1H NMR spectra show uncontrolled chemical shift variations between spectra in specific ppm areas, and peaks must be re-aligned before binning or statistical analysis of metabolomics data.

## When NOT to use

- Input spectra have not yet undergone ppm scale calibration or baseline correction — realignment is one step within the NMRProcFlow processing sequence that includes calibration of the ppm scale and base line correction
- The spectral region contains intertwined peaks and the spectra do not have the same amount of peaks in that region — Parametric Time Warping will not produce a good result in this case

## Inputs

- Set of calibrated 1D 1H NMR spectra loaded into NMRProcFlow (Bruker or Agilent/Varian FID, or nmrML format)
- User-selected ppm areas showing uncontrolled chemical shift variations between spectra

## Outputs

- Realigned spectra exported as a data matrix (e.g. aligned_spectra.csv) with resonances aligned across spectra within the selected ppm areas

## How to apply

Load the calibrated 1D 1H NMR spectra into NMRProcFlow's NMRspec application (R-based, with C++ processing algorithms in the Rnmr1D module via Rcpp) and inspect them overlaid or stacked in the NMR spectra viewer. Interactively select the set of ppm areas whose resonances show uncontrolled variations in chemical shifts between spectra. For each selected ppm area, apply the Least-Square realignment algorithm to correct the chemical shift shifts between spectra. For regions with intertwined peaks where Least-Square does not fit, apply Parametric Time Warping (Bloemberg et al. 2010) instead, which gives a better result provided all spectra contain the same number of peaks in that region. Export the realigned spectra as a data matrix and re-inspect in the spectra viewer to confirm alignment.

## Related tools

- **NMRProcFlow** (GUI application in which ppm areas are selected interactively and the Least-Square realignment is applied to spectra) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (The NMR spectra processing application of NMRProcFlow, implemented mainly in R, that performs the realignment of a set of ppm areas)
- **R** (Language in which the NMRspec spectra processing application is implemented)
- **Rcpp** (Package used to implement some processing algorithms in C++ within the Rnmr1D module)

## Examples

```
docker pull docker.io/nmrprocflow/npflow; sh ./npflow start  # then select ppm areas in the NMR spectra viewer and apply Least-Square realignment
```

## Evaluation signals

- The output data matrix exists with the same number of spectral rows as the input set of spectra
- On re-inspection in the NMR spectra viewer, resonances within the selected ppm areas are aligned across spectra
- For Parametric Time Warping regions, all spectra contain the same number of peaks in the selected ppm area (precondition for a good result)
- The realigned matrix is consistent with the preceding processing steps (ppm scale calibration, baseline correction) applied in NMRProcFlow

## Limitations

- Parametric Time Warping produces a better result only provided that all spectra have the same amount of peaks in the region with intertwined peaks
- Spectra alignment is a challenge that requires user expertise; automated batch-mode processing has not yet reached full maturity, so interactive selection of ppm areas in the spectra viewer is needed
- For crowded regions with heavy peak overlap, local signal deconvolution-based quantification is only planned in a future version of NMRProcFlow

## Evidence

- [intro] Least-Square realignment of selected ppm areas: "re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [supplementary] PTW for intertwined peaks: [evidence span withheld — anchor status: not_found]
- [supplementary] PTW precondition: [evidence span withheld — anchor status: not_found]
- [readme] processing scope: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
- [abstract] coverage: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
