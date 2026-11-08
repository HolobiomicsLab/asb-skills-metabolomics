---
name: nmr-voigt-lineshape-fitting
description: Use when processed 2D/pseudo-3D NMR spectra contain overlapping cross-peaks
  requiring per-peak quantitative parameters (amplitudes, widths, relaxation T1/T2,
  concentrations) that simple peak picking cannot resolve.
when_to_use_negative:
- The goal is coarse peak detection/coordinate listing only — use Simple Picker or
  DEEP Picker directly instead of full line-shape deconvolution
- The spectrum is 1D and requires only processing/projection viewing rather than quantitative
  cross-peak deconvolution
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3418
tools:
- name: COLMARvista
  role: Web-based integrated suite providing the Voigt/Gaussian deconvolution module,
    pseudo-3D series peak fitting, and interactive visualization of fitted vs. experimental
    spectra
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs the Voigt (Gaussian–Lorentzian convolution) line-shape fitting of
    each cross-peak, including jointly constrained fits for overlapping peaks
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: AI-driven deep neural network peak picking that supplies the cross-peak list
    used as input to the Voigt fitting routine
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:65ba2d7756adc9111962dfc91668cc1f376abf083d5514f38de4440efce93725
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# nmr-voigt-lineshape-fitting

## Summary

Fit 2D NMR cross-peaks with Voigt (Gaussian–Lorentzian convolution) line shapes to extract per-peak amplitude, center frequency, and width parameters, enabling quantitative analysis such as cross-peak-specific relaxation parameters even under significant peak overlap.

## When to use

Use when processed 2D/pseudo-3D NMR spectra contain overlapping cross-peaks requiring per-peak quantitative parameters (amplitudes, widths, relaxation T1/T2, concentrations) that simple peak picking cannot resolve.

## When NOT to use

- The goal is coarse peak detection/coordinate listing only — use Simple Picker or DEEP Picker directly instead of full line-shape deconvolution
- The spectrum is 1D and requires only processing/projection viewing rather than quantitative cross-peak deconvolution

## Inputs

- Processed 2D or pseudo-3D NMR spectrum (e.g. Bruker fid/ser processed in COLMARvista)
- Cross-peak coordinate list produced by automated deconvolution (DEEP Picker or Simple Picker)

## Outputs

- Per-peak fitted Voigt parameters: amplitude, center frequency, and width for each cross-peak
- Structured CSV output table with one row per picked cross-peak and converged fit parameters
- Automated reconstruction of the experimental spectrum from the fitted line shapes

## How to apply

Load the processed 2D NMR spectrum (e.g. Bruker `fid`/`ser` processed client-side in COLMARvista with phase correction, apodization, zero filling, solvent suppression, and baseline correction) together with the cross-peak coordinate list produced by automated deconvolution (DEEP Picker or local maxima Simple Picker). Select the Voigt Fitter module within the COLMARvista integrated suite and fit each cross-peak with a Voigt (Gaussian–Lorentzian convolution) line shape, estimating per-peak amplitude, center frequency, and width. Where peaks overlap, constrain the fit jointly so cross-peak-specific quantitative parameters are resolved for each overlapping component. For pseudo-3D series (relaxation T1/T2, titration, kinetics), perform multi-plane sequential peak fitting across the series planes; write the fitted per-peak parameters to a structured CSV output table.

## Related tools

- **COLMARvista** (Web-based integrated suite providing the Voigt/Gaussian deconvolution module, pseudo-3D series peak fitting, and interactive visualization of fitted vs. experimental spectra) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs the Voigt (Gaussian–Lorentzian convolution) line-shape fitting of each cross-peak, including jointly constrained fits for overlapping peaks) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (AI-driven deep neural network peak picking that supplies the cross-peak list used as input to the Voigt fitting routine) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- Fitted line shapes, when summed, reconstruct the experimental spectrum (automated reconstruction as described for the Voigt Fitter integration)
- The output CSV table contains exactly one row per picked cross-peak, each with converged fit parameters (amplitude, center, width)
- Overlapping peaks yield distinct, separately resolved quantitative parameters for each component after joint fitting
- Extracted cross-peak-specific relaxation parameters are consistent across planes of the pseudo-3D series fit

## Limitations

- Locally downloaded COLMARvista requires browser configuration (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false) for WebWorker/WebAssembly to load, which poses a security risk
- Fit quality depends on the accuracy of the upstream peak list; the workflow presumes the cross-peak coordinate list was produced by automated deconvolution first
- The README notes no changelog is available for tracking tool changes

## Evidence
