---
name: 2d-crosspeak-parameter-extraction
description: Use when a processed 2D NMR spectrum plus a picked cross-peak list needs
  quantitative per-peak parameters (relaxation, concentration), especially where peaks
  overlap and simple intensity readout is unreliable.
when_to_use_negative:
- The goal is only visual inspection of contours/projections — plain spectral viewing
  in COLMARvista (WebGL contours, 1D slices) suffices without line-shape fitting
- The task is CEST dip pre-analysis or exchange-residue filtering ('Show ≥2 Peaks
  Only') rather than cross-peak parameter extraction from a standard 2D spectrum
- Peaks are fully isolated and only qualitative peak positions are needed — Simple
  Picker alone may be sufficient without deconvolution
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Web-based (JavaScript/WebAssembly) platform hosting the Voigt Fitter module;
    loads the spectrum and peak list, runs the fits, and writes the per-peak CSV output
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: AI-driven deep neural network that produces the cross-peak list (automated
    deconvolution of complex 2D spectra) that seeds the Voigt fits
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs the Gaussian–Lorentzian (Voigt) line-shape fitting of each picked
    cross-peak, jointly constraining overlapping peaks, and reconstructs the fitted
    spectrum
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:c60ac9cd2b246f063abf721c169d3a84412d85b9784fef617b67be8056ac2104
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# 2D-crosspeak-parameter-extraction

## Summary

Extracts cross-peak-specific quantitative parameters (amplitude, center frequency, width) from 2D/pseudo-3D NMR spectra by fitting each picked peak with a Voigt (Gaussian–Lorentzian convolution) line shape in COLMARvista's integrated Voigt Fitter, resolving even significantly overlapped signals.

## When to use

Use when a processed 2D NMR spectrum plus a picked cross-peak list needs quantitative per-peak parameters (relaxation, concentration), especially where peaks overlap and simple intensity readout is unreliable.

## When NOT to use

- The goal is only visual inspection of contours/projections — plain spectral viewing in COLMARvista (WebGL contours, 1D slices) suffices without line-shape fitting
- The task is CEST dip pre-analysis or exchange-residue filtering ('Show ≥2 Peaks Only') rather than cross-peak parameter extraction from a standard 2D spectrum
- Peaks are fully isolated and only qualitative peak positions are needed — Simple Picker alone may be sufficient without deconvolution

## Inputs

- Processed 2D or pseudo-3D NMR spectrum (e.g., Bruker fid/ser processed in-browser, or NUS-reconstructed data)
- Cross-peak coordinate/peak list from automated deconvolution (DEEP Picker or Simple Picker)

## Outputs

- Per-peak fitted Voigt parameters (amplitude, center frequency, width) written as a structured CSV table
- Automated reconstruction of the experimental spectrum from fitted line shapes (fit-quality overlay)

## How to apply

Load the processed 2D NMR spectrum file together with the cross-peak coordinate list produced by automated deconvolution (DEEP Picker neural network or local-maxima Simple Picker), then select the Voigt Fitter module in COLMARvista's integrated suite. Fit each cross-peak with a Voigt line shape, estimating per-peak amplitude, center frequency, and width parameters. For overlapping peaks, constrain the fit jointly so cross-peak-specific quantitative parameters are resolved for each component rather than attributing intensity to a single peak. Write the fitted per-peak parameters to a structured CSV table. For pseudo-3D series (T1, T2 relaxation, titration, kinetics), apply multi-plane sequential peak fitting across the series planes; the fitted line shapes are used to automatically reconstruct the experimental spectrum as a fit-quality check.

## Related tools

- **COLMARvista** (Web-based (JavaScript/WebAssembly) platform hosting the Voigt Fitter module; loads the spectrum and peak list, runs the fits, and writes the per-peak CSV output) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (AI-driven deep neural network that produces the cross-peak list (automated deconvolution of complex 2D spectra) that seeds the Voigt fits) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs the Gaussian–Lorentzian (Voigt) line-shape fitting of each picked cross-peak, jointly constraining overlapping peaks, and reconstructs the fitted spectrum) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- Fitted line shapes reconstruct the experimental spectrum (automated reconstruction from the Voigt Fitter integration matches the raw contour plot)
- Output CSV contains exactly one row per picked cross-peak with converged fit parameters (amplitude, center, width)
- For known overlap cases, jointly constrained fits resolve distinct component parameters per cross-peak rather than merged values
- Cross-peak-specific relaxation parameters derived from pseudo-3D series fits behave monotonically/physically across the series planes (e.g., T1/T2 decay plots)

## Limitations

- Highly overlapped regions depend on the quality of the DEEP Picker deconvolution; mis-picked or missed peaks cannot be recovered by the fitter
- Running COLMARvista locally (opening index.html) requires modifying browser security settings for WebWorker/WebAssembly (e.g., Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false), which the README warns poses a security risk
- Processing and fitting occur client-side in the browser; very large spectra or peak lists may be constrained by browser/WebAssembly performance

## Evidence

- [abstract] Integration enables cross-peak-specific relaxation parameters for overlapping peaks: "allows the automated deconvolution and reconstruction of the experimental spectra for highly quantitative analysis, from compound concentration determination to the extraction of cross-peak specific"
- [abstract] COLMARvista integration with DEEP Picker and Voigt Fitter: "Its full integration with DEEP Picker and Voigt Fitter programs allows the automated deconvolution and reconstruction of the experimental spectra"
- [readme] README lists peak picking and 2D line-shape fitting capabilities: "**Peak Picking & 2D Line-Shape Fitting**: AI-driven DEEP Picker neural network, local maxima Simple Picker, and Voigt/Gaussian deconvolution."
- [readme] README describes pseudo-3D series peak fitting for relaxation parameters: "**Pseudo-3D Series Processing & 2D DOSY**: Multi-plane sequential peak fitting across relaxation ($T_1, T_2$), titration, and kinetics series"
- [readme] README documents browser configuration needed for local WebAssembly use: "This program uses WebWorker and WebAssembly, which cannot be loaded automatically when run locally unless you modify your browser settings."
