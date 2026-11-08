---
name: spectra-viewer-interactive-processing
description: Use when raw 1D NMR FID spectra (Bruker or Varian/Agilent, or nmrML)
  from metabolomics experiments need interactive visual inspection and processing
  steps that require expert visual judgement, such as baseline distortion assessment
  or alignment checks.
when_to_use_negative:
- When a fully automated, hands-off batch processing pipeline is required without
  expert visual supervision — the article notes batch-mode toolboxes usable by non-experts
  have not reached full maturity, and this skill is explicitly interactive
- When only a command-line interface is acceptable — R-based processing alone has
  a command-line barrier the authors designed NMRProcFlow's GUI to avoid
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRProcFlow
  role: Open source GUI application providing the interactive interface for 1D NMR
    spectra visualization and processing
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: NMR spectra processing application implemented in R (with C++ algorithms via
    Rcpp) behind the interactive GUI
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRviewer
  role: Client–server visualization module communicating with the processing module
    through AJAX web services
- name: Docker
  role: Deployment container for NMRProcFlow modules sharing a data volume; run via
    the npflow start/stop scripts
  repo: https://github.com/INRA/NMRProcFlow
- name: R Shiny
  role: Framework implementing the NMRspec GUI based on event-driven programming
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:d59f18c6f6bce693fcd51b027dfb215b0402f8a9fc32aeeba0c6b66f7b0fa9d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - Docker
  - NMRProcFlow
  - NMRspec
  - NMRviewer
  - R
  - R Shiny
  - Rcpp
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# spectra-viewer-interactive-processing

## Summary

Interactive processing of 1D NMR metabolomics spectra (1H & 13C) through the NMRProcFlow 'NMR spectra viewer', where users visually explore overlaid or stacked spectra while applying processing steps such as ppm-scale calibration, baseline correction (locally or fully), realignment of ppm areas, and binning.

## When to use

Use when raw 1D NMR FID spectra (Bruker or Varian/Agilent, or nmrML) from metabolomics experiments need interactive visual inspection and processing steps that require expert visual judgement, such as baseline distortion assessment or alignment checks.

## When NOT to use

- When a fully automated, hands-off batch processing pipeline is required without expert visual supervision — the article notes batch-mode toolboxes usable by non-experts have not reached full maturity, and this skill is explicitly interactive
- When only a command-line interface is acceptable — R-based processing alone has a command-line barrier the authors designed NMRProcFlow's GUI to avoid

## Inputs

- Raw 1D NMR FID spectra from Bruker or Agilent Technologies (Varian), including Bruker folder structure with 1r files
- 1D NMR spectra in nmrML format
- Experimental-factor level metadata for grouping/coloring spectra

## Outputs

- Interactively processed 1D NMR spectra (calibrated, baseline-corrected, realigned, binned)
- Visually verified spectra with flat baseline regions and preserved peak shapes/intensities
- Spectra visualizations overlaid or stacked and grouped by experimental factor level

## How to apply

Deploy NMRProcFlow via Docker (pull docker.io/nmrprocflow/npflow or build with 'sh ./npflow build') and start it with 'sh ./npflow start', then open the web GUI at http://<host>:<port>/npflow/. Load raw 1D NMR spectra (FID from Bruker or Agilent/Varian, including Bruker folder structure with 1r files, or nmrML) into the NMRspec application (R with C++ algorithms via Rcpp). In the NMR spectra viewer — the central tool of the application — inspect spectra overlaid or stacked, zoom on intensity scale, and group spectra by color based on their experimental factor level. Apply processing in the viewer: calibration of the ppm scale, baseline correction scoped 'locally' on delimited ppm zones or 'fully' across the whole ppm scale, realignment of a set of ppm areas, and binning (Intelligent/Adaptive Intelligent Binning, variable size, or fixed width), choosing among at least two methods per processing stage. The rationale is that automated batch-mode workflows usable by non-experts have not reached maturity, so interactive processing with visual verification is still the best approach; biologists and spectroscopists can interact by visualizing spectra alongside experimental-factor levels.

## Related tools

- **NMRProcFlow** (Open source GUI application providing the interactive interface for 1D NMR spectra visualization and processing) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (NMR spectra processing application implemented in R (with C++ algorithms via Rcpp) behind the interactive GUI) — https://github.com/INRA/NMRProcFlow
- **NMRviewer** (Client–server visualization module communicating with the processing module through AJAX web services)
- **Docker** (Deployment container for NMRProcFlow modules sharing a data volume; run via the npflow start/stop scripts) — https://github.com/INRA/NMRProcFlow
- **R Shiny** (Framework implementing the NMRspec GUI based on event-driven programming)

## Examples

```
docker pull docker.io/nmrprocflow/npflow; sh ./npflow start  # then open http://<your_local_host>:8080/npflow/
```

## Evaluation signals

- Corrected spectra show flat baseline regions while peak shapes and intensities are preserved in the viewer
- Spectra can be grouped by color according to their experimental factor levels in the viewer, confirming metadata linkage
- All intended processing stages (ppm calibration, baseline correction locally or fully, realignment of ppm areas, binning) are reflected in the output
- The application responds at http://<your_local_host>:<port>/npflow/ after 'sh ./npflow start' and sessions purge after the configured PURGESESSIONS duration
- Binning output avoids the peak-shift artifacts of equally sized bins when Adaptive Intelligent Binning is selected

## Limitations

- Spectra alignment still requires user expertise; multiple challenges in 1D spectra processing for high-throughput metabolomics remain unsolved
- Automated batch-mode processing usable by non-experts has not reached full maturity — interactive expert judgement remains necessary
- Quantification based on local signal deconvolution for crowded regions with heavy peak overlap is only planned for a future version
- Requires a recent Linux OS supporting Docker; session data is destroyed after the configured validity period (PURGESESSIONS, default 2 days) and ZIP uploads are capped (MAXZIPSIZE=400 Mo)

## Evidence

- [readme] The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application.: "The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application."
- [readme] the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or fully), the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width): "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas, and the binning (Intelligent, variable size or"
- [abstract] NMRProcFlow covers all spectra processing steps including baseline correction, chemical shift calibration and alignment: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
- [methods] The NMRspec GUI has been implemented using the ‘R shiny’ framework based on event-driven programming: "The NMRspec GUI has been implemented using the ‘R shiny’ (http://shiny.rstudio.com/) framework based on event-driven programming"
- [intro] the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode (regarded as a black-box) in order to be widely used by non-expert users: "the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode (regarded as a black-box) in order to be widely used by non-expert users or"
