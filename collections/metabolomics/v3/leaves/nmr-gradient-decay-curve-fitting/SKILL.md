---
name: nmr-gradient-decay-curve-fitting
description: Use when the input is a pseudo-3D NMR series (e.g., pulsed-field-gradient
  DOSY, or T1/T2/titration/kinetics series of 2D planes) and per-component parameters
  such as diffusion coefficient D must be extracted, especially when peaks overlap.
when_to_use_negative:
- The input is a single 2D spectrum rather than a multi-plane series — there is no
  gradient/relaxation dimension from which to extract decay curves.
- The analysis goal is CEST dip analysis or exchange-active residue filtering ('Show
  ≥2 Peaks Only') rather than series fitting — COLMARvista provides a separate CEST/ChemEx
  workflow for that.
- Only processing/visualization of one plane is needed; the WebAssembly pipeline (phase
  correction, apodization, zero filling, solvent suppression, baseline correction)
  alone suffices without the sequential fitting step.
tools:
- name: COLMARvista
  role: Web-based platform that hosts the WebAssembly processing pipeline, the multi-plane
    sequential peak fitting across the pseudo-3D gradient series, the Stejskal-Tanner
    decay-plot display, and export of fitted D values to a table file.
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: AI-driven deep neural network peak picker used to pick cross-peaks in each
    2D plane of the gradient series, including overlapped signals, before deconvolution.
- name: Voigt Fitter
  role: Performs the Voigt/Gaussian line-shape deconvolution of each plane so that
    per-component cross-peak intensities can be traced across gradient strengths for
    decay fitting.
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:6a4826440adf294ab37dd5db0e18ae2230f659c9e2a4deb21647e9952b9a5fb7
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nmr-gradient-decay-curve-fitting

## Summary

Extracts per-cross-peak diffusion coefficients (D) from a pseudo-3D DOSY dataset — a series of 2D spectra acquired at increasing pulsed-field-gradient strengths — by multi-plane sequential peak fitting in COLMARvista and fitting each component's intensity decay to the Stejskal-Tanner relation.

## When to use

Use when the input is a pseudo-3D NMR series (e.g., pulsed-field-gradient DOSY, or T1/T2/titration/kinetics series of 2D planes) and per-component parameters such as diffusion coefficient D must be extracted, especially when peaks overlap.

## When NOT to use

- The input is a single 2D spectrum rather than a multi-plane series — there is no gradient/relaxation dimension from which to extract decay curves.
- The analysis goal is CEST dip analysis or exchange-active residue filtering ('Show ≥2 Peaks Only') rather than series fitting — COLMARvista provides a separate CEST/ChemEx workflow for that.
- Only processing/visualization of one plane is needed; the WebAssembly pipeline (phase correction, apodization, zero filling, solvent suppression, baseline correction) alone suffices without the sequential fitting step.

## Inputs

- Raw Bruker pseudo-3D DOSY dataset (fid/ser time-domain files): a series of 2D spectra acquired at different pulsed-field-gradient strengths
- Optionally, a reference peak list for assignment transfer (.list, .tab, .txt, .csv)

## Outputs

- Per-component pulsed-field-gradient Stejskal-Tanner decay curves (interactive decay plots)
- Table file of fitted per-component diffusion coefficients D

## How to apply

Load the pseudo-3D dataset — a series of 2D planes acquired at different pulsed-field-gradient strengths — into COLMARvista and process each plane through the client-side WebAssembly pipeline (automated phase correction, apodization, zero filling, solvent suppression, baseline correction). Apply peak picking (DEEP Picker or Simple Picker) followed by Voigt/Gaussian line-shape deconvolution so that overlapped cross-peaks are resolved into components with quantitative intensities. Perform multi-plane sequential peak fitting across the gradient series to obtain peak-specific intensity decay curves as a function of gradient strength. Fit each component's decay to the pulsed-field-gradient Stejskal-Tanner relation to extract D, inspect the interactive decay plots for mono-exponential behavior, and save the fitted per-component diffusion coefficients to a table file.

## Related tools

- **COLMARvista** (Web-based platform that hosts the WebAssembly processing pipeline, the multi-plane sequential peak fitting across the pseudo-3D gradient series, the Stejskal-Tanner decay-plot display, and export of fitted D values to a table file.) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (AI-driven deep neural network peak picker used to pick cross-peaks in each 2D plane of the gradient series, including overlapped signals, before deconvolution.)
- **Voigt Fitter** (Performs the Voigt/Gaussian line-shape deconvolution of each plane so that per-component cross-peak intensities can be traced across gradient strengths for decay fitting.)

## Evaluation signals

- Fitted D values and their decay-plot curves are consistent with mono-exponential Stejskal-Tanner behavior for each resolved component, as judged by the interactive decay plot fit quality.
- Each component yields a peak-specific intensity decay curve defined over the full gradient-strength series (multi-plane sequential peak fitting completed for all planes).
- Reconstructed (deconvoluted) spectra match the experimental spectra, indicating the DEEP Picker + Voigt Fitter deconvolution underlies the quantitative extraction correctly.
- The output table file contains one diffusion coefficient per resolved cross-peak component, matching the picked peak list.

## Limitations

- The series-fitting mechanism for overlapped signals is described but not separately validated by the article; validation is limited to checking Stejskal-Tanner consistency in the interactive decay plots.
- Local (offline) use requires browser configuration for WebWorker/WebAssembly support (e.g., Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari local file restrictions disabled), which the README warns poses a security risk.
- The web server is static and accepts no uploads — all data stays local, so processing must be done client-side in the browser.

## Evidence
