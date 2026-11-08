---
name: spectral-realignment-of-ppm-areas
description: Use when uncontrolled chemical-shift variations between 1D 1H NMR spectra
  cause the same resonances to appear at shifted ppm positions across samples, and
  peaks must be re-aligned within selected ppm areas before binning or quantification.
when_to_use_negative:
- Input is already a bucket/intensity data matrix (post-binning) — realignment operates
  on spectra, not on the final table
- Spectra show no uncontrolled chemical-shift variations between samples, so re-alignment
  would add no correction
- Goal is 2D NMR or non-metabolomics spectra processing — NMRProcFlow is dedicated
  to 1D spectra (1H & 13C) for NMR-based metabolomics
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: NMRProcFlow
  role: Open-source GUI software that provides the interactive workflow in which the
    realignment of a set of ppm areas is applied to 1D NMR metabolomics spectra
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: The NMR spectra processing application (R Shiny GUI) whose spectra viewer
    lets the user visually verify re-aligned ppm areas before binning
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: Docker image 'npflow' (built or pulled from docker.io/nmrprocflow/npflow)
    packages and runs the NMRspec/NMRProcFlow modules used for realignment
  repo: https://github.com/INRA/NMRProcFlow
- name: Rcpp
  role: C++ processing algorithms behind the Rnmr1D module execute the spectra processing
    computations, including re-alignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:7a65710b205868b4de06dcec75afee24ec64fa19ac9a5664c60298be067faae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - NMRspec
  - R
  - R Shiny
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-realignment-of-ppm-areas

## Summary

Re-aligns selected ppm areas across a set of 1D 1H NMR metabolomics spectra to correct uncontrolled chemical-shift variations between spectra, as one stage of the NMRProcFlow interactive processing chain before binning.

## When to use

Use when uncontrolled chemical-shift variations between 1D 1H NMR spectra cause the same resonances to appear at shifted ppm positions across samples, and peaks must be re-aligned within selected ppm areas before binning or quantification.

## When NOT to use

- Input is already a bucket/intensity data matrix (post-binning) — realignment operates on spectra, not on the final table
- Spectra show no uncontrolled chemical-shift variations between samples, so re-alignment would add no correction
- Goal is 2D NMR or non-metabolomics spectra processing — NMRProcFlow is dedicated to 1D spectra (1H & 13C) for NMR-based metabolomics

## Inputs

- Set of processed 1D 1H NMR spectra imported from raw FID files (Bruker or Agilent/Varian, or Bruker 1r files) in NMRspec
- Ppm-calibrated and baseline-corrected spectra
- User-selected ppm areas (interval definitions) to re-align

## Outputs

- Re-aligned spectra in which resonances within the selected ppm areas are coherent across the whole spectra set
- Aligned spectra feeding downstream binning (intelligent, variable size or fixed width) and the final bucket/intensity matrix

## How to apply

Import the raw 1D 1H FID dataset (Bruker or Agilent/Varian format, or Bruker 1r files) into NMRspec via the 'npflow' Docker image so spectra are processed by the Rnmr1D module (R with C++ algorithms via Rcpp). First calibrate the ppm scale and apply baseline correction (locally or fully) so that shifts are measured on corrected spectra. Then interactively select the ppm areas to re-align, using the NMR spectra viewer (R Shiny GUI) to overlay or stack spectra and zoom to check that corresponding resonances coincide after re-alignment. Note that for each spectra processing stage at least two methods can be applied, so the realignment method is a decision point chosen per dataset. Perform realignment before binning (Adaptive Intelligent Binning, variable size or fixed width), since misaligned peaks would otherwise be split or smeared across bin edges; Adaptive Intelligent Binning largely circumvents loss of information and peak-shift artifacts of equally sized bins.

## Related tools

- **NMRProcFlow** (Open-source GUI software that provides the interactive workflow in which the realignment of a set of ppm areas is applied to 1D NMR metabolomics spectra) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (The NMR spectra processing application (R Shiny GUI) whose spectra viewer lets the user visually verify re-aligned ppm areas before binning) — https://github.com/INRA/NMRProcFlow
- **Docker** (Docker image 'npflow' (built or pulled from docker.io/nmrprocflow/npflow) packages and runs the NMRspec/NMRProcFlow modules used for realignment) — https://github.com/INRA/NMRProcFlow
- **Rcpp** (C++ processing algorithms behind the Rnmr1D module execute the spectra processing computations, including re-alignment)

## Examples

```
docker pull docker.io/nmrprocflow/npflow; sh ./npflow start
```

## Evaluation signals

- Overlaid or stacked spectra in the NMR spectra viewer show resonances within the selected ppm areas coinciding across all spectra after re-alignment
- Re-alignment was applied as a distinct stage between baseline correction/ppm calibration and binning, matching the documented processing order
- Downstream binning produces a bucket/intensity matrix (one intensity column per spectrum, one row per ppm bucket) without peak-shift artifacts at bin edges
- The re-aligned spectra set remains traceable through the exported CSV table from the NMRProcFlow session

## Limitations

- Spectra alignment requires user expertise and is cited as one of the unsolved challenges in 1D NMR spectra processing for high-throughput metabolomics
- Automated batch-mode processing workflows usable by non-experts have not reached full maturity; interactive processing with a spectra viewer is still the best approach
- Realignment only corrects resonances within the ppm areas the user explicitly selects for re-alignment

## Evidence

- [abstract] covers all spectra processing steps including baseline correction, chemical shift calibration and alignment: "covers all spectra processing steps including baseline correction, chemical shift calibration and alignment."
- [intro] re-alignment of resonances having uncontrolled variations in chemical shifts between spectra: "removal of resonance signals of solvents and other contaminants, re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [methods] the realignment of a set of ppm areas: "the base line correction (locally or fully), the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
- [results] For each spectra processing stage, at least two methods can be applied: "Indeed, for each spectra processing stage, at least two methods can be applied (see Supplementary Information S1)."
- [readme] NMRProcFlow open source software provides an efficient GUI tool for spectra processing from 1D NMR metabolomics data: "The NMRProcFlow open source software provides an efficient GUI tool for spectra processing from 1D NMR metabolomics data, based on an interactive interface for the spectra visualization, that greatly"
