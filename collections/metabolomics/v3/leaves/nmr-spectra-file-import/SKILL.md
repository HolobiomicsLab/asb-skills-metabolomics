---
name: nmr-spectra-file-import
description: Use when raw 1D 1H/13C NMR FID data from Bruker or Agilent/Varian instruments
  (Bruker folder structure with 1r files), or spectra in nmrML format, must be loaded
  into NMRProcFlow before baseline correction, alignment, or binning.
when_to_use_negative:
- Input is already a processed feature/bucket table (e.g. baseline_corrected_spectra.csv)
  rather than raw FID or nmrML spectra — import is only the first workflow step
- Data are 2D NMR spectra — NMRProcFlow is dedicated to 1D NMR (1H & 13C) spectra
  processing
- No Docker-capable Linux environment is available and local installation cannot be
  performed
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NMRProcFlow
  role: GUI application whose NMR spectra viewer accepts imported 1D vendor FID and
    nmrML data for interactive processing
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: The NMR spectra processing application (implemented in R with an R Shiny GUI)
    that receives and renders the imported spectra
- name: Docker
  role: Container runtime hosting the npflow image with its shared data volume through
    which spectra files are imported and shared across modules
  repo: docker.io/nmrprocflow/npflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:c551595c9df0a521fbea10eed77f27d31113613eb4e7b4dcc77e2c7adaadf7bd
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nmr-spectra-file-import

## Summary

Loads raw 1D NMR data — FID files from Bruker and Agilent (Varian) instruments, including the Bruker folder structure with 1r files, as well as nmrML-format spectra — into NMRProcFlow's interactive processing workspace so subsequent steps (ppm calibration, baseline correction, realignment, binning) can be applied.

## When to use

Use when raw 1D 1H/13C NMR FID data from Bruker or Agilent/Varian instruments (Bruker folder structure with 1r files), or spectra in nmrML format, must be loaded into NMRProcFlow before baseline correction, alignment, or binning.

## When NOT to use

- Input is already a processed feature/bucket table (e.g. baseline_corrected_spectra.csv) rather than raw FID or nmrML spectra — import is only the first workflow step
- Data are 2D NMR spectra — NMRProcFlow is dedicated to 1D NMR (1H & 13C) spectra processing
- No Docker-capable Linux environment is available and local installation cannot be performed

## Inputs

- Raw Free Induction Decay (FID) files from Bruker instruments (Bruker folder structure with 1r files)
- Raw FID files from Agilent Technologies (Varian) instruments
- Spectra in nmrML format

## Outputs

- Imported 1D 1H/13C spectra rendered in NMRProcFlow's NMR spectra viewer (overlaid or stacked, zoomable, grouped by experimental-factor level color)
- Session data persisted under the shared Docker data volume (DATASETS, default /opt/data) available to all processing modules

## How to apply

Deploy NMRProcFlow as the 'npflow' Docker image (pulled from docker.io/nmrprocflow/npflow or built locally with 'sh ./npflow build') on a Linux host supporting Docker, then start it with 'sh ./npflow start' and access it at http://<your_local_host>:8080/npflow/. Upload or point the application at raw FID data: the current version accepts FID from Bruker GmbH and Agilent Technologies (Varian), including the Bruker folder structure with 1r files, and also supports the nmrML format. Data are stored in a shared Docker data volume configured via the DATASETS setting (default /opt/data) so all modules communicate through AJAX web services. Decision point on format: prefer nmrML for cross-platform interoperability; vendor FID with Bruker folder structure is accepted natively. Rationale: NMRProcFlow is an open source GUI tool whose NMR spectra viewer is the central application, so correct import is verified by seeing the spectra rendered overlaid or stacked with zoomable intensity scale and factor-level color grouping.

## Related tools

- **NMRProcFlow** (GUI application whose NMR spectra viewer accepts imported 1D vendor FID and nmrML data for interactive processing) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (The NMR spectra processing application (implemented in R with an R Shiny GUI) that receives and renders the imported spectra)
- **Docker** (Container runtime hosting the npflow image with its shared data volume through which spectra files are imported and shared across modules) — docker.io/nmrprocflow/npflow

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- Bruker (folder structure with 1r files) and Agilent/Varian FID inputs load without vendor-specific import errors
- Imported spectra are visible in the NMR spectra viewer overlaid or stacked, with intensity-scale zoom and factor-level color grouping
- The 'npflow' Docker image is present (docker image listing shows the repository tag) and the app answers at http://<your_local_host>:8080/npflow/
- Session data persist in the DATASETS directory (default /opt/data) and are reachable by all modules via the shared data volume
- nmrML files are accepted alongside the two vendor FID formats

## Limitations

- Only FID from two major vendors (Bruker GmbH and Agilent Technologies (Varian)) are currently accepted; other vendor formats may require conversion
- Requires a recent Linux OS that supports Docker; the web app runs on an HTTP port (default 8080) configured in ./etc/npflow.conf
- Sessions are purged after a validity period (PURGESESSIONS=2 days from last change) and uploads are capped (MAXZIPSIZE=400 Mo)

## Evidence

- [results] Vendor FID formats accepted: "The current version of NMRProcFlow accepts the Free Induction Decay (FID) coming from two major vendors namely Bruker GmbH & Agilent Technologies (Varian)"
- [results] nmrML format support: "Besides, we already support the nmrML format (http://nmrml.org/, Rocca-Serra et al. 2016)."
- [readme] Docker deployment route: "You can simply pull the docker image from docker hub: $ docker pull docker.io/nmrprocflow/npflow"
- [readme] Spectra viewer verification: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color"
- [intro] Tool purpose: "we have been developing NMRProcFlow, an open source software that greatly helps spectra processing (Fig. 1)."
