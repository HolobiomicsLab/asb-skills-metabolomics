---
name: ppm-zone-delimitation
description: Use when raw 1D NMR metabolomics spectra (Bruker FID, Varian/Agilent
  FID, or nmrML) show baseline distortion or macromolecule residuals and you must
  delimit ppm areas for local baseline correction or realignment in NMRProcFlow.
when_to_use_negative:
- Spectra already carry a flat baseline and only need downstream steps such as ppm-scale
  calibration, realignment, or binning — apply those steps directly instead
- The user wants a fully automated batch-mode black-box workflow without interactive
  inspection; NMRProcFlow's delimitation is deliberately interactive GUI-driven
- The data are not 1D NMR metabolomics spectra (e.g. feature tables or 2D NMR); NMRProcFlow's
  viewer and processing stages target 1D spectra
edam_operation: http://edamontology.org/operation_3216
tools:
- name: NMRProcFlow
  role: Open source GUI application whose NMR spectra viewer is the central tool for
    interactively exploring overlaid/stacked spectra and delimiting ppm zones for
    correction
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: The NMR spectra processing application (implemented mainly in R with an R
    Shiny GUI) that executes the locally or fully scoped baseline correction on the
    delimited zones
- name: Rcpp
  role: Package through which C++ implementations of the processing algorithms (including
    baseline correction) are invoked from the R backend
- name: Docker
  role: Runtime environment used to deploy the NMRProcFlow modules (via the npflow
    image) for running the interactive processing session
  repo: https://github.com/INRA/NMRProcFlow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:182b5e9d2f3176576af88ba4a1861b4d6d304cdd2516657e7b599cb4fb30502d
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# ppm-zone-delimitation

## Summary

Delimiting specific ppm zones in 1D NMR metabolomics spectra within NMRProcFlow's interactive viewer so that scoped processing steps—especially local baseline correction and realignment of ppm areas—can be applied only where needed, e.g. to eliminate residual macromolecule effects without distorting peaks elsewhere.

## When to use

Use when raw 1D NMR metabolomics spectra (Bruker FID, Varian/Agilent FID, or nmrML) show baseline distortion or macromolecule residuals and you must delimit ppm areas for local baseline correction or realignment in NMRProcFlow.

## When NOT to use

- Spectra already carry a flat baseline and only need downstream steps such as ppm-scale calibration, realignment, or binning — apply those steps directly instead
- The user wants a fully automated batch-mode black-box workflow without interactive inspection; NMRProcFlow's delimitation is deliberately interactive GUI-driven
- The data are not 1D NMR metabolomics spectra (e.g. feature tables or 2D NMR); NMRProcFlow's viewer and processing stages target 1D spectra

## Inputs

- Raw 1D 1H or 13C NMR FID files from Bruker or Agilent Technologies (Varian), including the Bruker folder structure with 1r files
- nmrML-format spectra files
- User-delineated ppm zone selections made in the interactive NMR spectra viewer

## Outputs

- Baseline-corrected 1D NMR spectra with flat baseline regions in the delimited ppm zones (e.g. exportable as baseline_corrected_spectra.csv)
- Visually verified corrected spectra overlaid/stacked in the NMR spectra viewer

## How to apply

Load the raw FID spectra into NMRProcFlow (NMRspec backend, implemented in R with C++ algorithms via Rcpp/Rnmr1D) and inspect them overlaid or stacked in the 'NMR spectra viewer', the central tool of the application. Interactively zoom on the intensity scale and visually identify baseline distortion, then delimit the ppm areas to which the correction applies. Choose between a 'locally' scoped correction restricted to the selected ppm zones and a 'fully' scoped correction across the whole ppm scale; local correction eliminates residual effects from macromolecules in extracts and reduces the dominance of an intense peak over less intense neighboring ones. Re-inspect the viewer to confirm flat baseline regions while peak shapes and intensities are preserved, then export the corrected spectra. Rationale: NMRProcFlow was built by involving NMR spectroscopists and offers at least two methods per processing stage, so interactive delimitation in the GUI is the intended way to exercise expert judgment rather than opaque batch processing.

## Related tools

- **NMRProcFlow** (Open source GUI application whose NMR spectra viewer is the central tool for interactively exploring overlaid/stacked spectra and delimiting ppm zones for correction) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (The NMR spectra processing application (implemented mainly in R with an R Shiny GUI) that executes the locally or fully scoped baseline correction on the delimited zones)
- **Rcpp** (Package through which C++ implementations of the processing algorithms (including baseline correction) are invoked from the R backend)
- **Docker** (Runtime environment used to deploy the NMRProcFlow modules (via the npflow image) for running the interactive processing session) — https://github.com/INRA/NMRProcFlow

## Examples

```
sh ./npflow start
```

## Evaluation signals

- Corrected spectra show flat baseline regions in the delimited ppm zones when re-inspected in the NMR spectra viewer
- Peak shapes and peak intensities within and outside the delimited zones are preserved (no clipping or distortion of neighboring peaks)
- The chosen scope ('locally' vs 'fully') is reflected in the processing: only the selected ppm areas change for local correction
- The processed/exported spectra (e.g. baseline_corrected_spectra.csv) can be re-loaded and overlaid against the raw spectra for a visual before/after check
- The correction is performed through at least one of the >=2 methods available per processing stage in NMRProcFlow, matching the documented workflow

## Limitations

- 1D spectra processing often requires an expert eye for disentangling the intertwined peaks, so zone delimitation quality depends on user expertise; spectra alignment in particular requires user expertise
- Local baseline correction targets residual effects from macromolecules in extracts and intense peaks masking less intense neighbors; its effectiveness for other baseline artifacts is not established in the article
- Running the interactive tool requires a recent Linux OS that supports Docker, and sessions have a validity duration (PURGESESSIONS default 2 days) with a max ZIP size (MAXZIPSIZE=400 Mo)

## Evidence

- [abstract] abstract: "dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
- [methods] methods: "some processing algorithms have been implemented in C + + using the Rcpp package (Eddelbuettel and Francois 2011)"
- [readme] readme: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
