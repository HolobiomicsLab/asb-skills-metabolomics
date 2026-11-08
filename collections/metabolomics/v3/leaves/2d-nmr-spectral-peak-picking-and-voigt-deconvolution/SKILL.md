---
name: 2d-nmr-spectral-peak-picking-and-voigt-deconvolution
description: Use when 2D or pseudo-3D NMR spectra contain overlapped cross-peaks and
  quantitative per-cross-peak intensities are required, e.g. for concentration determination
  or cross-peak-specific relaxation/DOSY parameter extraction.
when_to_use_negative:
- The spectra contain only well-resolved, non-overlapped cross-peaks where simple
  peak integration suffices and deconvolution adds no quantitative benefit
- The input is already a processed peak/assignment list to be matched against reference
  lists (.list, .tab, .txt, .csv) — that is the automated assignment transfer workflow,
  not peak picking/deconvolution
- The goal is CEST dip pre-analysis and ChemEx exchange-parameter fitting rather than
  line-shape deconvolution of 2D spectra
edam_operation: http://edamontology.org/operation_3221
tools:
- name: COLMARvista
  role: Web-based platform hosting the client-side WebAssembly processing pipeline
    and integrating peak picking and deconvolution into an interactive 2D/pseudo-3D
    viewer with multi-plane sequential fitting
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: Deep neural network for automated, accurate peak picking/deconvolution of
    complex 2D NMR spectra, including overlapped cross-peaks
- name: Voigt Fitter
  role: Performs Voigt/Gaussian line-shape fitting that underlies automated deconvolution,
    spectrum reconstruction, and quantitative per-cross-peak parameter extraction
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:50fd7d62a2cd0921c35c02916839e47026f0dcdc8b00dc800c7e72701fcdeabc
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - DEEP Picker
  - Voigt Fitter
  edam_operation: http://edamontology.org/operation_3221
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# 2D NMR spectral peak picking and Voigt deconvolution

## Summary

COLMARvista integrates AI-driven DEEP Picker (or local-maxima Simple Picker) peak picking with Voigt/Gaussian line-shape deconvolution and automated spectrum reconstruction, enabling highly quantitative per-cross-peak analysis of 2D and pseudo-3D NMR data even when peaks overlap.

## When to use

Use when 2D or pseudo-3D NMR spectra contain overlapped cross-peaks and quantitative per-cross-peak intensities are required, e.g. for concentration determination or cross-peak-specific relaxation/DOSY parameter extraction.

## When NOT to use

- The spectra contain only well-resolved, non-overlapped cross-peaks where simple peak integration suffices and deconvolution adds no quantitative benefit
- The input is already a processed peak/assignment list to be matched against reference lists (.list, .tab, .txt, .csv) — that is the automated assignment transfer workflow, not peak picking/deconvolution
- The goal is CEST dip pre-analysis and ChemEx exchange-parameter fitting rather than line-shape deconvolution of 2D spectra

## Inputs

- Raw Bruker time-domain FID data (fid, ser) for 2D or pseudo-3D series
- Processed 2D spectra / pseudo-3D planes (e.g. gradient-strength or relaxation series)
- Pseudo-3D NUS data for SMILE reconstruction prior to picking

## Outputs

- Picked peak lists per 2D plane (DEEP Picker or Simple Picker)
- Voigt/Gaussian-deconvoluted per-cross-peak intensities and reconstructed spectra
- Peak-specific intensity decay curves across a pseudo-3D series
- Per-component parameter tables (e.g. diffusion coefficients D, cross-peak-specific relaxation parameters)

## How to apply

Load raw Bruker time-domain FID data (fid, ser) into COLMARvista, which processes each 2D plane client-side via WebAssembly with automated phase correction, apodization, zero filling, solvent suppression, and baseline correction. Pick peaks on the processed spectra using the AI-driven DEEP Picker neural network, or the local-maxima Simple Picker as a lighter alternative. Apply Voigt/Gaussian deconvolution so that COLMARvista's integration with DEEP Picker and Voigt Fitter automatically deconvolves and reconstructs the experimental spectra, separating overlapped signals into individual components. For pseudo-3D series (relaxation T1/T2, titration, kinetics, or pulsed-field-gradient DOSY), perform multi-plane sequential peak fitting across planes to extract peak-specific intensity decay curves. Feed those per-cross-peak intensity curves into downstream parameter fitting (e.g. the Stejskal-Tanner relation for diffusion coefficients D). Judge success by agreement between reconstructed and experimental spectra and by whether resolved components give consistent, physically expected decay behavior in the interactive decay plots.

## Related tools

- **COLMARvista** (Web-based platform hosting the client-side WebAssembly processing pipeline and integrating peak picking and deconvolution into an interactive 2D/pseudo-3D viewer with multi-plane sequential fitting) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (Deep neural network for automated, accurate peak picking/deconvolution of complex 2D NMR spectra, including overlapped cross-peaks)
- **Voigt Fitter** (Performs Voigt/Gaussian line-shape fitting that underlies automated deconvolution, spectrum reconstruction, and quantitative per-cross-peak parameter extraction)

## Evaluation signals

- The reconstructed spectrum from deconvolution visually matches the experimental 2D spectrum (automated deconvolution and reconstruction check)
- Overlapped signals are resolved into distinct components whose fitted positions/intensities are stable across the series planes in multi-plane sequential peak fitting
- Extracted per-peak intensity decay curves show the expected mono-exponential Stejskal-Tanner behavior for DOSY, judged in the interactive decay plots
- Fitted per-component parameters (e.g. D values) are consistent across replicates/planes and land in physically reasonable ranges
- Picking is reproducible when DEEP Picker and Simple Picker are compared on well-separated peaks

## Limitations

- The series-fitting mechanism built on DEEP Picker/Voigt Fitter integration is described but not separately validated in the article for the pseudo-3D parameter-extraction workflow
- Local offline use (opening index.html) requires per-browser configuration of WebWorker/WebAssembly access (e.g. --allow-file-access-from-files in Chrome, security.fileuri.strict_origin_policy=false in Firefox), which the README flags as a security risk
- NUS reconstruction depends on the proprietary SMILE WebAssembly plugin, included only under a linking exception with permission from its original author
- No changelog was found for the repository

## Evidence
