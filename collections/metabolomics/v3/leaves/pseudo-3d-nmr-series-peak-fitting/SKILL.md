---
name: pseudo-3d-nmr-series-peak-fitting
description: Use when a pseudo-3D NMR dataset (a series of 2D planes at varying gradient
  strength, relaxation delay, titration point, or time point) needs per-cross-peak
  intensities and decay-parameter fitting, especially with overlapping peaks.
when_to_use_negative:
- Input is a single 1D or plain 2D spectrum with no series dimension — multi-plane
  sequential fitting is not applicable
- Cross-peaks are fully resolved, so direct integration suffices and deconvolution-based
  per-component fitting adds no value
- Analysis goal is outside the supported series types (T1/T2 relaxation, titration,
  kinetics, DOSY), e.g., pure CEST profiling handled by the separate CEST/ChemEx workflow
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: COLMARvista
  role: Web-based platform hosting the WebAssembly processing pipeline and the multi-plane
    sequential peak fitting / DOSY diffusion fitting with interactive decay plots
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: Deep-neural-network peak picking used to identify cross-peaks in each 2D plane
    prior to deconvolution
- name: Voigt Fitter
  role: Performs Voigt/Gaussian line-shape deconvolution of overlapped cross-peaks
    to yield per-component intensities across the series
- name: SMILE WebAssembly plugin
  role: Iterative NUS reconstruction of pseudo-3D series before peak picking when
    data are non-uniformly sampled
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:6601c17e51e8834ea53c747423328bfacb8d3a7d865c6ffeb9409f5ff45ff302
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - DEEP Picker
  - SMILE WebAssembly plugin
  - Voigt Fitter
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 21
---

# pseudo-3d-nmr-series-peak-fitting

## Summary

Per-cross-peak parameter extraction from pseudo-3D NMR series (DOSY gradient arrays, T1/T2 relaxation, titration, kinetics) by combining COLMARvista's client-side plane processing with DEEP Picker/Voigt Fitter deconvolution and multi-plane sequential peak fitting.

## When to use

Use when a pseudo-3D NMR dataset (a series of 2D planes at varying gradient strength, relaxation delay, titration point, or time point) needs per-cross-peak intensities and decay-parameter fitting, especially with overlapping peaks.

## When NOT to use

- Input is a single 1D or plain 2D spectrum with no series dimension — multi-plane sequential fitting is not applicable
- Cross-peaks are fully resolved, so direct integration suffices and deconvolution-based per-component fitting adds no value
- Analysis goal is outside the supported series types (T1/T2 relaxation, titration, kinetics, DOSY), e.g., pure CEST profiling handled by the separate CEST/ChemEx workflow

## Inputs

- pseudo-3D NMR dataset (series of 2D spectra, e.g., DOSY pulsed-field-gradient series) as raw Bruker `fid`/`ser` files
- NUS pseudo-3D data requiring SMILE reconstruction (optional)

## Outputs

- per-cross-peak fitted parameters (e.g., diffusion coefficient D) saved to a table file
- interactive per-component decay plots from the multi-plane sequential fit

## How to apply

Load the pseudo-3D series (raw Bruker `fid`/`ser` files) into COLMARvista and process each 2D plane through the client-side WebAssembly pipeline — automated phase correction, apodization, zero filling, solvent suppression, and baseline correction; use SMILE reconstruction for NUS pseudo-3D data. Pick peaks with the AI-driven DEEP Picker (or local-maxima Simple Picker) and apply Voigt/Gaussian line-shape deconvolution so that overlapped cross-peaks are resolved into individual components. Then run multi-plane sequential peak fitting across the series to extract peak-specific intensity decay curves as a function of the series variable (e.g., pulsed-field-gradient strength). For DOSY, fit each component's decay to the Stejskal-Tanner relation to obtain the diffusion coefficient D, displayed as interactive decay plots, and save the per-component D values to a table file. Judge success by whether the fitted D values and decay-plot curves are consistent with the expected mono-exponential Stejskal-Tanner behavior for each resolved component.

## Related tools

- **COLMARvista** (Web-based platform hosting the WebAssembly processing pipeline and the multi-plane sequential peak fitting / DOSY diffusion fitting with interactive decay plots) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (Deep-neural-network peak picking used to identify cross-peaks in each 2D plane prior to deconvolution)
- **Voigt Fitter** (Performs Voigt/Gaussian line-shape deconvolution of overlapped cross-peaks to yield per-component intensities across the series)
- **SMILE WebAssembly plugin** (Iterative NUS reconstruction of pseudo-3D series before peak picking when data are non-uniformly sampled)

## Evaluation signals

- Fitted D values (or T1/T2 parameters) follow mono-exponential Stejskal-Tanner (or relaxation) decay for each resolved component
- Interactive decay-plot fit quality confirms per-component curve consistency across the gradient/delay series
- Automated spectral reconstruction from the deconvolution matches the processed experimental spectra plane by plane
- Output table file contains one fitted parameter per cross-peak component across all series planes

## Limitations

- The series-fitting mechanism is described but not separately validated in the article; correctness rests on the deconvolution/reconstruction quality of DEEP Picker and Voigt Fitter
- Overlapped peaks depend on the accuracy of AI-driven picking and Voigt/Gaussian deconvolution, which may fail for severely crowded spectra
- Local (offline) use requires modifying browser settings to enable WebWorker/WebAssembly file access, which poses a documented security risk

## Evidence
