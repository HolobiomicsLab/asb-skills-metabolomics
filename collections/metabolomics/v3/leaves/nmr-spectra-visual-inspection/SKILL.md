---
name: nmr-spectra-visual-inspection
description: Use when raw 1D NMR spectra (FID from Bruker or Agilent/Varian) need
  to be visually explored — e.g. to locate a reference resonance for ppm calibration,
  check alignment, or compare spectra grouped by experimental factor.
when_to_use_negative:
- When the goal is a fully automated, hands-off batch pipeline — the article notes
  batch-mode workflows usable by non-experts have not reached full maturity, but this
  skill is deliberately interactive
- When input is already a processed/binned feature table rather than raw or processed
  spectra to inspect
- When processing 2D NMR spectra — NMRProcFlow is dedicated to 1D spectra (1H & 13C)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRProcFlow
  role: Open-source GUI application whose central 'NMR spectra viewer' supports interactive
    visual inspection of 1D NMR spectra
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: Spectra processing application (R/R Shiny) hosting the interactive viewer
    session used for inspection
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: Distributes the 'npflow' image (docker.io/nmrprocflow/npflow) that packages
    the application sharing a data volume for the spectra
- name: R Shiny
  role: Framework underlying the NMRspec event-driven GUI through which visual inspection
    is performed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:5a435599a5c68beb7267288646c52b4a404fd0081fc4e3de026094fc3c4254ed
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# nmr-spectra-visual-inspection

## Summary

Interactive visual inspection of 1D NMR spectra (1H & 13C) using the NMRProcFlow 'NMR spectra viewer', where spectra are overlaid or stacked and color-grouped by factor level before and during processing decisions such as chemical shift calibration.

## When to use

Use when raw 1D NMR spectra (FID from Bruker or Agilent/Varian) need to be visually explored — e.g. to locate a reference resonance for ppm calibration, check alignment, or compare spectra grouped by experimental factor.

## When NOT to use

- When the goal is a fully automated, hands-off batch pipeline — the article notes batch-mode workflows usable by non-experts have not reached full maturity, but this skill is deliberately interactive
- When input is already a processed/binned feature table rather than raw or processed spectra to inspect
- When processing 2D NMR spectra — NMRProcFlow is dedicated to 1D spectra (1H & 13C)

## Inputs

- Raw 1D 1H or 13C NMR FID files from Bruker GmbH or Agilent Technologies (Varian), including the Bruker folder structure with 1r files
- nmrML-format spectra files (supported by NMRProcFlow)
- Experimental-factor level metadata for color-grouping spectra

## Outputs

- On-screen overlaid or stacked spectra views with zoomed regions and factor-level color groups
- Validated processing outputs such as calibrated_spectra.csv (e.g. spectra after ppm-scale calibration)

## How to apply

Launch NMRProcFlow from the 'npflow' Docker image with raw spectra in the shared data volume (DATASETS, default /opt/data), then load them into an NMRspec session opened at http://<your_local_host>:<port>/npflow/. In the 'NMR spectra viewer' — the central tool and core of the application — display spectra overlaid or stacked, zoom on the intensity scale, and group sets of spectra by color based on their factor level to link experimental design to the spectra. Inspect the region containing the target resonance (e.g. the reference peak for ppm-scale calibration) so that interactive processing decisions are made with expert visual judgment rather than blind batch automation. Because NMRspec is implemented in R with R Shiny and Rcpp-accelerated algorithms, heavy operations stay responsive while inspection remains interactive. Export only the outcomes you intended (e.g. calibrated_spectra.csv) after visually confirming the reference resonance sits at its target ppm position on every spectrum.

## Related tools

- **NMRProcFlow** (Open-source GUI application whose central 'NMR spectra viewer' supports interactive visual inspection of 1D NMR spectra) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (Spectra processing application (R/R Shiny) hosting the interactive viewer session used for inspection) — https://github.com/INRA/NMRProcFlow
- **Docker** (Distributes the 'npflow' image (docker.io/nmrprocflow/npflow) that packages the application sharing a data volume for the spectra)
- **R Shiny** (Framework underlying the NMRspec event-driven GUI through which visual inspection is performed)

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start  # then inspect spectra at http://<your_local_host>:8080/npflow/
```

## Evaluation signals

- Re-inspecting the spectra in the viewer after calibration shows the reference resonance of every spectrum sitting at the target ppm position on the calibrated axis
- Spectra can be overlaid or stacked and zoomed on the intensity scale without rendering errors in the viewer
- Spectra are correctly color-grouped by their factor level, matching the supplied experimental-factor metadata
- The exported output (e.g. calibrated_spectra.csv) exists and contains only the intended processing outcome (no baseline correction, realignment, or binning applied)

## Limitations

- Inspection is inherently interactive and expert-driven: the article notes automated batch-mode processing for non-experts has not reached full maturity and that spectra alignment in particular requires user expertise
- Vendor and format coverage is limited to Bruker and Agilent (Varian) FID plus nmrML; other formats are not accepted
- Docker sessions are purged after PURGESESSIONS (default 2 days) since the last change, and uploads are limited by MAXZIPSIZE (default 400 Mo)

## Evidence

- [readme] The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application.: "The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application."
- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color"
- [intro] NMRProcFlow was built by involving NMR spectroscopists eager to have a quick and easy tool to use.: "It was built by involving NMR spectroscopists eager to have a quick and easy tool to use"
- [methods] The NMRspec GUI has been implemented using the 'R shiny' framework based on event-driven programming.: "The NMRspec GUI has been implemented using the ‘R shiny’ (http://shiny.rstudio.com/) framework based on event-driven programming"
