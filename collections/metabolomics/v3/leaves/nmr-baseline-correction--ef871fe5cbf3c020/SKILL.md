---
name: nmr-baseline-correction
description: Use when raw 1D 1H NMR spectra (Bruker or Agilent/Varian FID, or Bruker
  1r files) show baseline distortion from macromolecules in extracts or an intense
  peak dominating less intense neighboring ones, before alignment or binning.
when_to_use_negative:
- Input is already a bucket/intensity matrix (post-binning) — baseline correction
  operates on spectra, and buckets failing the signal-to-noise filter are handled
  at the output step instead
- Data are 2D NMR spectra — NMRProcFlow is dedicated to 1D (1H & 13C) spectra processing
- The processing need is deconvolution-based quantification in crowded regions with
  heavy peak overlap — that method is only planned in a future version of NMRProcFlow
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: NMRProcFlow
  role: Open-source GUI software whose processing chain includes the base line correction
    (locally or fully) step within 1D NMR metabolomics spectra processing
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: The NMR spectra processing application (R Shiny GUI) in which baseline correction
    is applied; some algorithms implemented in C++ via Rcpp
  repo: https://github.com/INRA/NMRProcFlow
- name: Docker
  role: Runs NMRProcFlow modules embedded in Docker images (the 'npflow' image) that
    share a data volume and communicate through AJAX web services
  repo: docker.io/nmrprocflow/npflow
- name: Rcpp
  role: R package used to implement C++ processing algorithms (e.g. baseline correction)
    invoked by the Rnmr1D module
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:e376940bf2695222ed7cde7870c2a3f50edc6a22e7f54ec6f875ad08ec8b0092
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
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nmr-baseline-correction

## Summary

Apply baseline correction to 1D NMR metabolomics spectra (1H & 13C) within NMRProcFlow's processing chain, choosing either a 'locally' scoped correction on selected ppm zones or a 'fully' scoped correction across the whole ppm scale.

## When to use

Use when raw 1D 1H NMR spectra (Bruker or Agilent/Varian FID, or Bruker 1r files) show baseline distortion from macromolecules in extracts or an intense peak dominating less intense neighboring ones, before alignment or binning.

## When NOT to use

- Input is already a bucket/intensity matrix (post-binning) — baseline correction operates on spectra, and buckets failing the signal-to-noise filter are handled at the output step instead
- Data are 2D NMR spectra — NMRProcFlow is dedicated to 1D (1H & 13C) spectra processing
- The processing need is deconvolution-based quantification in crowded regions with heavy peak overlap — that method is only planned in a future version of NMRProcFlow

## Inputs

- Raw 1D 1H NMR FID data from Bruker or Agilent Technologies (Varian)
- Bruker folder structure with 1r files
- User-delimited ppm zones for locally scoped correction
- nmrML format files (supported by NMRProcFlow)

## Outputs

- Baseline-corrected spectra (e.g. exported as baseline_corrected_spectra.csv)
- Visually verified spectra with flat baseline regions and preserved peak shapes/intensities

## How to apply

Load the raw FID dataset into NMRProcFlow's NMRspec application (implemented in R with C++ algorithms via the Rcpp Rnmr1D module, run from the 'npflow' Docker image). In the interactive 'NMR spectra viewer', inspect overlaid or stacked spectra to identify baseline distortion and, for local correction, delimit the ppm areas the correction should apply to. Apply the base line correction choosing either a 'locally' scoped correction on the selected ppm zones or a 'fully' scoped correction across the whole ppm scale. The rationale is to eliminate residual effects due to macromolecules in extracts and reduce the prevalence of an intense peak on less intense neighboring ones. Visually verify in the viewer that corrected spectra show flat baseline regions while peak shapes and intensities are preserved, then export (e.g. baseline_corrected_spectra.csv) before proceeding to chemical shift calibration, alignment, and binning.

## Related tools

- **NMRProcFlow** (Open-source GUI software whose processing chain includes the base line correction (locally or fully) step within 1D NMR metabolomics spectra processing) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (The NMR spectra processing application (R Shiny GUI) in which baseline correction is applied; some algorithms implemented in C++ via Rcpp) — https://github.com/INRA/NMRProcFlow
- **Docker** (Runs NMRProcFlow modules embedded in Docker images (the 'npflow' image) that share a data volume and communicate through AJAX web services) — docker.io/nmrprocflow/npflow
- **Rcpp** (R package used to implement C++ processing algorithms (e.g. baseline correction) invoked by the Rnmr1D module)

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- Corrected spectra show flat baseline regions when inspected in the NMR spectra viewer (overlaid or stacked)
- Peak shapes and intensities are preserved after correction
- Correction scope matches the choice: 'locally' only in the delimited ppm zones, or 'fully' across the whole ppm scale
- The corrected spectra feed the downstream chain (ppm calibration, realignment, binning) and appear in the exported CSV matrix with one intensity column per spectrum and one row per ppm bucket

## Limitations

- For each spectra processing stage, at least two methods can be applied, and spectra alignment requires user expertise; interactive processing with a spectra viewer remains the best approach rather than fully automatic batch mode
- Baseline correction in targeted metabolomics is local and aims to eliminate residual effects of macromolecules in extracts — heavily overlapped peaks requiring signal deconvolution are not covered
- Purged sessions and working-space settings (e.g. DATASETS=/opt/data, MAXZIPSIZE=400 Mo, HTTP_PORT=8080) constrain how the Docker-based application is operated

## Evidence

- [abstract] NMRProcFlow covers all spectra processing steps including baseline correction, chemical shift calibration and alignment: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction, chemical shift calibration"
- [intro] Baseline correction is one of the processing stages for which at least two methods can be applied: "those encountered during the various stages of processing (baseline correction, chemical shift calibration, removal of resonance signals of solvents and other contaminants"
- [methods] The base line correction can be applied locally or fully across the ppm scale: "the base line correction (locally or fully)"
- [methods] Local baseline correction eliminates residual effects of macromolecules in extracts and reduces the prevalence of intense peaks on less intense neighbors: [evidence span withheld — anchor status: not_found]
- [readme] The README lists baseline correction among the spectra processing steps of the application: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas"
