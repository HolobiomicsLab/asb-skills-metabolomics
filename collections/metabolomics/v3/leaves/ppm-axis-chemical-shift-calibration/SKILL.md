---
name: ppm-axis-chemical-shift-calibration
description: Use when raw 1D NMR spectra (Bruker or Agilent/Varian FID, or Bruker
  1r files) show uncontrolled chemical shift offsets and a reference resonance must
  be pinned to its target ppm value prior to baseline correction, alignment, or binning.
when_to_use_negative:
- Input is already a processed bucket/intensity data matrix — calibration operates
  on raw spectra, not on post-binning tables
- Spectra need only binning or baseline correction and already share a correct ppm
  axis — calibration would add no value
- Fully automated batch processing without expert visual inspection is mandatory and
  no reference resonance can be identified interactively — the article notes batch-mode
  workflows usable by non-experts have not reached full maturity
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: NMRProcFlow
  role: GUI software providing the interactive ppm-scale calibration step within its
    1D NMR spectra processing workflow
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: R Shiny application inside NMRProcFlow in which spectra are loaded, inspected
    in the spectra viewer, and calibrated
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: 'Deployment vehicle: the ''npflow'' Docker image (docker.io/nmrprocflow/npflow)
    shares a data volume (DATASETS, /opt/data) with the web modules'
  repo: docker.io/nmrprocflow/npflow
- name: R Shiny
  role: Framework implementing the NMRspec GUI used for event-driven, interactive
    calibration
- name: Rcpp
  role: Package through which some processing algorithms underlying the workflow are
    implemented in C++ within Rnmr1D
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:282c030b9f1bedc14ed7beee09bbb9317785ac00c882c8320bd661cf89570b23
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ppm-axis-chemical-shift-calibration

## Summary

Interactive chemical shift calibration of the ppm scale for 1D 1H (and 13C) NMR spectra using NMRProcFlow's NMRspec GUI, so a reference resonance is set to its target ppm position across a spectra set before further processing.

## When to use

Use when raw 1D NMR spectra (Bruker or Agilent/Varian FID, or Bruker 1r files) show uncontrolled chemical shift offsets and a reference resonance must be pinned to its target ppm value prior to baseline correction, alignment, or binning.

## When NOT to use

- Input is already a processed bucket/intensity data matrix — calibration operates on raw spectra, not on post-binning tables
- Spectra need only binning or baseline correction and already share a correct ppm axis — calibration would add no value
- Fully automated batch processing without expert visual inspection is mandatory and no reference resonance can be identified interactively — the article notes batch-mode workflows usable by non-experts have not reached full maturity

## Inputs

- Raw 1D 1H (or 13C) NMR FID files in Bruker or Agilent Technologies (Varian) format, or Bruker folder structure with 1r files
- nmrML-format spectra files (also supported by NMRProcFlow)
- Target ppm value of the reference resonance for calibration

## Outputs

- Calibrated spectra with the reference resonance at the target ppm position on the ppm axis
- Exported calibrated spectra table (e.g. calibrated_spectra.csv) if calibration-only processing is requested

## How to apply

Launch NMRProcFlow from the 'npflow' Docker image (docker pull docker.io/nmrprocflow/npflow) with raw spectra in the shared data volume (DATASETS, default /opt/data). Load the FID-derived spectra into an NMRspec session (R Shiny GUI backed by the Rnmr1D module using Rcpp-implemented C++ algorithms) and inspect them overlaid or stacked in the 'NMR spectra viewer', zooming on the region containing the reference resonance. Apply the ppm-scale calibration step interactively so the reference peak is set to its target chemical shift in every spectrum; NMRProcFlow offers at least two methods per processing stage (see Supplementary Information S1). Rationale: interactive calibration with a spectra viewer is preferred over black-box batch processing because alignment/shift issues require user expertise. Export the calibrated spectra alone if calibration is the only required step, or continue to baseline correction (locally or fully), removal of solvent/contaminant resonance zones, re-alignment of ppm areas, and binning.

## Related tools

- **NMRProcFlow** (GUI software providing the interactive ppm-scale calibration step within its 1D NMR spectra processing workflow) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (R Shiny application inside NMRProcFlow in which spectra are loaded, inspected in the spectra viewer, and calibrated) — https://github.com/INRA/NMRProcFlow
- **Docker** (Deployment vehicle: the 'npflow' Docker image (docker.io/nmrprocflow/npflow) shares a data volume (DATASETS, /opt/data) with the web modules) — docker.io/nmrprocflow/npflow
- **R Shiny** (Framework implementing the NMRspec GUI used for event-driven, interactive calibration)
- **Rcpp** (Package through which some processing algorithms underlying the workflow are implemented in C++ within Rnmr1D)

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start  # then calibrate the ppm scale interactively at http://<your_local_host>:<port>/npflow/
```

## Evaluation signals

- calibrated_spectra.csv (or equivalent export) exists after the calibration-only run
- Re-inspection in the 'NMR spectra viewer' shows the reference resonance of every spectrum sitting at the target ppm position on the calibrated axis
- No baseline correction, realignment, or binning artifacts present when calibration was requested as the sole processing stage
- Calibrated axis values across the spectra set are consistent (same reference peak position), enabling subsequent re-alignment of ppm areas
- Session artifacts persist per npflow.conf settings (e.g. PURGESESSIONS validity window, DATASETS=/opt/data working space)

## Limitations

- Calibration is interactive and requires an expert eye; automated batch-mode workflows usable by non-experts have not reached full maturity
- Spectra alignment (and by extension shift correction) remains an unsolved challenge requiring user expertise (Larive et al. 2015)
- Application must be run via the npflow Docker image on a Linux OS supporting Docker; sessions expire after PURGESESSIONS (default 2 days) and ZIP uploads are limited to MAXZIPSIZE=400 Mo

## Evidence

- [abstract] NMRProcFlow covers all spectra processing steps including baseline correction, chemical shift calibration and alignment: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
- [methods] the spectra processing includes: the calibration of the ppm scale: "the spectra processing includes: the calibration of the ppm scale"
- [methods] NMRspec GUI implemented using R shiny: "The NMRspec GUI has been implemented using the ‘R shiny’ (http://shiny.rstudio.com/) framework based on event-driven programming"
- [readme] README processing steps including ppm calibration: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
- [intro] interactive viewer preferred over batch mode: "the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode (regarded as a black-box) in order to be widely used by non-expert users or"
