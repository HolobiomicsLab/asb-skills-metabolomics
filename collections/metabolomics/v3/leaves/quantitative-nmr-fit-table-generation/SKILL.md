---
name: quantitative-nmr-fit-table-generation
description: Use when quantitative per-cross-peak parameters (amplitude, center frequency,
  width) or cross-peak-specific relaxation values from T1/T2 pseudo-3D series are
  needed from 2D NMR spectra, especially where peaks overlap.
when_to_use_negative:
- The goal is only visual inspection or contour viewing of the spectrum without quantitative
  per-peak parameters — COLMARvista's 1D/2D/3D WebGL viewing suffices
- The spectrum contains well-isolated peaks where simple local-maxima Simple Picker
  suffices and joint deconvolution of overlaps is unnecessary
- The analysis goal is NUS reconstruction or SMILE iterative reconstruction of raw
  time-domain data rather than line-shape fitting of picked peaks
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Web-based platform hosting the line-shape fitting workflow; provides the Voigt
    Fitter module, pseudo-3D series multi-plane peak fitting, and CSV output of fitted
    parameters
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: Deep neural network that automatically picks/deconvolves cross-peaks in complex
    2D spectra, supplying the peak list that drives the Voigt fitting
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs Voigt (Gaussian–Lorentzian) line-shape fitting of each cross-peak,
    jointly resolving overlapping components and reconstructing the spectrum
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:fa61027f181ab98f2a7945017f6a1a65a3a3245365d0f4ce0ea3b4040a6bfdd1
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# quantitative-nmr-fit-table-generation

## Summary

Fit each picked cross-peak in a 2D or pseudo-3D NMR spectrum with a Voigt (Gaussian–Lorentzian convolution) line shape to extract quantitative per-peak parameters — amplitude, center frequency, and width — including for overlapping peaks, and write them to a structured fit table.

## When to use

Use when quantitative per-cross-peak parameters (amplitude, center frequency, width) or cross-peak-specific relaxation values from T1/T2 pseudo-3D series are needed from 2D NMR spectra, especially where peaks overlap.

## When NOT to use

- The goal is only visual inspection or contour viewing of the spectrum without quantitative per-peak parameters — COLMARvista's 1D/2D/3D WebGL viewing suffices
- The spectrum contains well-isolated peaks where simple local-maxima Simple Picker suffices and joint deconvolution of overlaps is unnecessary
- The analysis goal is NUS reconstruction or SMILE iterative reconstruction of raw time-domain data rather than line-shape fitting of picked peaks

## Inputs

- Processed 2D or pseudo-3D NMR spectrum (Bruker time-domain fid/ser processed in-browser, or already-transformed spectra)
- Cross-peak coordinate list from automated deconvolution (DEEP Picker output; supports .list, .tab, .txt, .csv reference lists)

## Outputs

- CSV table of fitted per-peak Voigt parameters (amplitude, center frequency, width) with one row per picked cross-peak
- Reconstructed spectrum from the fitted line shapes (for fit-validation against the experimental spectrum)

## How to apply

Load the processed 2D NMR spectrum (e.g., Bruker fid/ser processed client-side in COLMARvista with phase correction, apodization, zero filling, solvent suppression, and baseline correction) together with the cross-peak coordinate list produced by DEEP Picker. Select the Voigt Fitter module within COLMARvista's integrated suite and fit each cross-peak with a Voigt line shape, estimating per-peak amplitude, center frequency, and width parameters. Where peaks overlap, constrain the fit jointly so that cross-peak-specific quantitative parameters are resolved for each overlapping component; for pseudo-3D relaxation (T1, T2), titration, or kinetics series, apply multi-plane sequential peak fitting across the series planes. Write the fitted per-peak parameters to a structured output table in CSV format, with one row per picked cross-peak. Validate by confirming the fitted line shapes reconstruct the experimental spectrum, as the DEEP Picker/Voigt Fitter integration is designed for automated deconvolution and reconstruction for highly quantitative analysis.

## Related tools

- **COLMARvista** (Web-based platform hosting the line-shape fitting workflow; provides the Voigt Fitter module, pseudo-3D series multi-plane peak fitting, and CSV output of fitted parameters) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (Deep neural network that automatically picks/deconvolves cross-peaks in complex 2D spectra, supplying the peak list that drives the Voigt fitting) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs Voigt (Gaussian–Lorentzian) line-shape fitting of each cross-peak, jointly resolving overlapping components and reconstructing the spectrum) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- The fitted line shapes reconstruct the experimental spectrum (automated reconstruction as described for the Voigt Fitter integration)
- The output CSV table contains one row per picked cross-peak with converged fit parameters (amplitude, center frequency, width)
- Overlapping peaks yield distinct cross-peak-specific parameters for each component rather than a single merged fit
- For pseudo-3D relaxation series, fitted amplitudes across planes support extraction of cross-peak-specific relaxation parameters (T1, T2)
- Compound concentration determination from fitted amplitudes is consistent with the quantitative-analysis use case described for the integration

## Limitations

- The browser-based program uses WebWorker and WebAssembly, which cannot be loaded automatically when run locally unless browser settings are modified (e.g., --allow-file-access-from-files in Chrome, security.fileuri.strict_origin_policy=false in Firefox), which poses a security risk
- Local offline use requires downloading the program and opening index.html with those browser modifications; otherwise use the hosted static web server
- Accurate joint fitting of heavily overlapping signals depends on the quality of the DEEP Picker deconvolution of the input spectrum

## Evidence
