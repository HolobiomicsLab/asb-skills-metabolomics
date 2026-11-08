---
name: parametric-time-warping-of-spectral-regions
description: Use when overlaid 1D 1H NMR spectra show uncontrolled chemical shift
  variations in a ppm area containing intertwined peaks, and Least-Square realignment
  cannot resolve the shifts.
when_to_use_negative:
- The ppm area shows peak shifts but peaks are not intertwined — use NMRProcFlow's
  Least-Square realignment algorithm instead
- Spectra have different amounts of peaks in the selected region — PTW only produces
  a better result provided all spectra have the same amount of peaks there
- Input is 2D NMR spectra — NMRProcFlow is dedicated to 1D spectra processing (1H
  & 13C)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_1319
tools:
- name: NMRProcFlow
  role: GUI application providing the interactive spectra viewer in which ppm areas
    are selected and the PTW alignment option is applied
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: R-based NMR spectra processing application hosting the processing algorithms
    that execute the realignment
- name: Rcpp
  role: R package through which processing algorithms such as warping are implemented
    in C++ within Rnmr1D
- name: Docker
  role: Deployment vehicle; the npflow image is pulled from docker.io/nmrprocflow/npflow
    and run to host the tool
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:3eeb02840dfc00c2b35b5ae9504ae53c7f9857950709214cfb7f07debb080c33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - NMRspec
  - R
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_1319
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# parametric-time-warping-of-spectral-regions

## Summary

Applies Parametric Time Warping (Bloemberg et al. 2010), implemented in NMRProcFlow's Rnmr1D module (C++ via Rcpp), to re-align resonances in 1D 1H NMR ppm areas whose peaks are intertwined and where the Least-Square realignment does not fit.

## When to use

Use when overlaid 1D 1H NMR spectra show uncontrolled chemical shift variations in a ppm area containing intertwined peaks, and Least-Square realignment cannot resolve the shifts.

## When NOT to use

- The ppm area shows peak shifts but peaks are not intertwined — use NMRProcFlow's Least-Square realignment algorithm instead
- Spectra have different amounts of peaks in the selected region — PTW only produces a better result provided all spectra have the same amount of peaks there
- Input is 2D NMR spectra — NMRProcFlow is dedicated to 1D spectra processing (1H & 13C)

## Inputs

- Set of calibrated 1D 1H NMR spectra (FID from Bruker or Agilent/Varian, or nmrML format) loaded into NMRProcFlow
- User-selected ppm area containing intertwined peaks with uncontrolled chemical shift variations between spectra

## Outputs

- Realigned spectra as a data matrix (e.g. aligned_spectra.csv) with one spectral row per input spectrum
- Warped ppm area inspectable in the NMR spectra viewer with resonances aligned across spectra

## How to apply

Load the calibrated 1D 1H NMR spectra into the NMRspec application of NMRProcFlow and inspect them overlaid/stacked in the NMR spectra viewer, grouping spectra by factor level to spot uncontrolled chemical shift variations. Select the ppm area whose resonances are intertwined. First attempt the Least-Square realignment algorithm; where it does not fit because peaks are intertwined, switch to Parametric Time Warping (Bloemberg et al. 2010) for that area. PTW will produce a better result provided that all spectra have the same amount of peaks in the selected region, so verify peak count consistency across spectra before warping. Export the realigned spectra as a data matrix (e.g. aligned_spectra.csv) and re-inspect the ppm area in the NMR spectra viewer to confirm the resonances now line up across spectra.

## Related tools

- **NMRProcFlow** (GUI application providing the interactive spectra viewer in which ppm areas are selected and the PTW alignment option is applied) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (R-based NMR spectra processing application hosting the processing algorithms that execute the realignment)
- **Rcpp** (R package through which processing algorithms such as warping are implemented in C++ within Rnmr1D)
- **Docker** (Deployment vehicle; the npflow image is pulled from docker.io/nmrprocflow/npflow and run to host the tool)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git; docker pull docker.io/nmrprocflow/npflow; sh ./npflow start
```

## Evaluation signals

- All spectra have the same amount of peaks in the selected ppm region (precondition for a good PTW result)
- Re-inspection in the NMR spectra viewer shows resonances within the selected ppm areas aligned across spectra
- Output data matrix exists with the same number of spectral rows as the input set of spectra
- Aligned chemical shifts remain confined to the originally selected ppm area, with no peaks displaced outside it
- Warping was applied only after the Least-Square adaptation was judged not to fit the intertwined-peak region

## Limitations

- PTW gives a better result only provided that all spectra have the same amount of peaks in the targeted region; differing peak counts undermine the warping
- 1D spectra processing for high-throughput metabolomics still has unsolved challenges; spectra alignment in particular requires user expertise
- Crowded regions with heavy peak overlap may need local signal deconvolution, which is only planned for a future version of NMRProcFlow
- Automated batch-mode processing remains a black box; interactive selection of ppm areas in the viewer is the supported route

## Evidence

- [other] PTW precondition: [evidence span withheld — anchor status: not_found]
- [intro] Re-alignment scope: "re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [readme] Processing stages in the GUI: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
- [methods] C++ implementation route: "some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011)"
- [abstract] Processing coverage: [evidence span withheld — anchor status: not_found]
