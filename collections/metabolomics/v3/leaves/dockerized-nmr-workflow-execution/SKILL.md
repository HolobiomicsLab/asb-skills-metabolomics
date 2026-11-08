---
name: dockerized-nmr-workflow-execution
description: Use when raw 1D NMR FID data (Bruker or Agilent/Varian, or nmrML) must
  be processed through calibration, baseline correction, realignment, and binning
  via NMRProcFlow deployed as a local Docker web application.
when_to_use_negative:
- Input is already a processed feature/bucket table rather than raw 1D NMR FID or
  nmrML spectra
- The goal is a fully automated, non-interactive batch pipeline — the article states
  batch-mode black-box workflows 'have not yet reached full maturity' and interactive
  processing is preferred
- Data are 2D NMR spectra — NMRProcFlow is dedicated to 1D (1H & 13C) spectra processing
edam_operation: http://edamontology.org/operation_3216
tools:
- name: NMRProcFlow
  role: GUI tool orchestrating the full 1D NMR spectra-processing workflow (ppm calibration,
    baseline correction, realignment, binning) via the npflow Docker image
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: 'Deployment container: modules are embedded in Docker images sharing a data
    volume and communicating through AJAX web services'
  repo: docker.io/nmrprocflow/npflow
- name: NMRspec
  role: NMR spectra-processing application (R) backing the interactive processing
    inside the running application
  repo: https://github.com/INRA/NMRProcFlow
- name: R Shiny
  role: Framework implementing the NMRspec GUI with event-driven programming for interactive
    spectra viewing and processing
- name: Rcpp
  role: C++ implementation of some processing algorithms within NMRspec
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:23edca26c3caa35576ca91226b878d0e31f780de3999998984656690954a420c
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# dockerized-nmr-workflow-execution

## Summary

Run the NMRProcFlow 1D NMR (1H & 13C) metabolomics spectra-processing workflow interactively inside the 'npflow' Docker image, with raw spectra staged in the shared /opt/data volume and the NMR spectra viewer as the central working interface.

## When to use

Use when raw 1D NMR FID data (Bruker or Agilent/Varian, or nmrML) must be processed through calibration, baseline correction, realignment, and binning via NMRProcFlow deployed as a local Docker web application.

## When NOT to use

- Input is already a processed feature/bucket table rather than raw 1D NMR FID or nmrML spectra
- The goal is a fully automated, non-interactive batch pipeline — the article states batch-mode black-box workflows 'have not yet reached full maturity' and interactive processing is preferred
- Data are 2D NMR spectra — NMRProcFlow is dedicated to 1D (1H & 13C) spectra processing

## Inputs

- Raw 1D NMR FID files from Bruker or Agilent Technologies (Varian), including the Bruker folder structure with 1r files
- nmrML-format spectra files
- Configuration file ./etc/npflow.conf (HTTP_PORT, DATASETS, PURGESESSIONS, MAXZIPSIZE, CORES, USRCONMGR)

## Outputs

- Processed 1D spectra (ppm-calibrated, baseline-corrected, realigned) accessible in the NMR spectra viewer session
- Binned/filtered bucket tables (e.g. Intelligent, variable size or fixed width binning), optionally filtered by signal-to-noise ratio
- Session data stored under the shared DATASETS volume (default /opt/data)

## How to apply

Clone the INRA/NMRProcFlow repository from GitHub and either build the image with `sh ./npflow build` or pull the prebuilt image with `docker pull docker.io/nmrprocflow/npflow`. Edit ./etc/npflow.conf to set HTTP_PORT (default 8080) and DATASETS (default /opt/data), the shared data volume where the raw spectra dataset must be placed, and session parameters such as PURGESESSIONS and MAXZIPSIZE. Start the application with `sh ./npflow start` and open http://<your_local_host>:<port>/npflow/, where the NMRspec application (implemented mainly in R with the R Shiny framework and C++ algorithms via Rcpp) provides the interactive spectra viewer for overlaid/stacked inspection. Apply each spectra-processing stage in the viewer — calibration of the ppm scale, baseline correction (locally or fully), realignment of ppm areas, and binning (Intelligent, variable size or fixed width) — choosing among at least two available methods per stage, since the article notes batch-mode black-box processing has not matured and interactive inspection with a spectroscopist's eye is still the best approach. Stop the application with `sh ./npflow stop`; the rationale for Dockerization is that the modules are embedded in Docker images sharing one data volume and communicating through AJAX web services, so no local R installation is needed.

## Related tools

- **NMRProcFlow** (GUI tool orchestrating the full 1D NMR spectra-processing workflow (ppm calibration, baseline correction, realignment, binning) via the npflow Docker image) — https://github.com/INRA/NMRProcFlow
- **Docker** (Deployment container: modules are embedded in Docker images sharing a data volume and communicating through AJAX web services) — docker.io/nmrprocflow/npflow
- **NMRspec** (NMR spectra-processing application (R) backing the interactive processing inside the running application) — https://github.com/INRA/NMRProcFlow
- **R Shiny** (Framework implementing the NMRspec GUI with event-driven programming for interactive spectra viewing and processing)
- **Rcpp** (C++ implementation of some processing algorithms within NMRspec)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && cd nmrproc && docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- The application is reachable at http://<your_local_host>:<port>/npflow/ after `sh ./npflow start` (default HTTP_PORT=8080)
- The raw spectra dataset is visible to all modules under the shared DATASETS volume (default /opt/data)
- Spectra can be visually explored overlaid or stacked with zoom on intensity scale in the NMR spectra viewer
- After ppm-scale calibration, the reference resonance sits at its target ppm position across all spectra
- Exported bucket/spectra tables reflect the chosen methods for each stage, with signal-to-noise-based bucket filtering applied where selected

## Limitations

- Requires a recent Linux OS supporting Docker; the image is large (~1.449 GB in the README example)
- Session validity is limited (PURGESESSIONS=2 days counted from last change) and ZIP uploads capped (MAXZIPSIZE=400 Mo)
- Only vendor FID from Bruker and Agilent (Varian), the Bruker 1r folder structure, and nmrML are accepted; some advanced quantification (local signal deconvolution for crowded regions) is only planned for a future version

## Evidence

- [abstract] NMRProcFlow covers all processing steps including baseline correction, chemical shift calibration: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
- [methods] Dockerized architecture sharing a data volume with AJAX web services: "each of them embedded in Docker images, sharing a data volume and communicating together through AJAX web services"
- [readme] Docker image pull command from the README: "You can simply pull the docker image from docker hub: $ docker pull docker.io/nmrprocflow/npflow"
- [readme] README listing of the processing stages available in the tool: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or fully), the realignment of a set of ppm areas, and the binning"
- [intro] Interactive processing preferred over batch mode: "the implementation of 1D NMR spectra processing workflows into a toolbox and operating automatically in batch mode (regarded as a black-box) in order to be widely used by non-expert users or"
