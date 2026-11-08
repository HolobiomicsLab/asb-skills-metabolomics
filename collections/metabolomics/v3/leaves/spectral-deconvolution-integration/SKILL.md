---
name: spectral-deconvolution-integration
description: Use when 2D or pseudo-3D NMR spectra contain overlapping cross-peaks
  requiring quantitative per-peak parameters (amplitude, width, T1/T2 relaxation,
  concentration) that simple peak picking cannot resolve.
when_to_use_negative:
- The spectrum is well-resolved with no peak overlap — a local-maxima Simple Picker
  suffices without joint Voigt deconvolution.
- The goal is purely qualitative spectral viewing or NUS reconstruction (SMILE) rather
  than quantitative per-peak parameter extraction.
- The data are 1D spectra where the 2D cross-peak fitting machinery and pseudo-3D
  series fitting are not applicable.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Web-based integrated suite hosting the Voigt/Gaussian 2D line-shape fitting
    workflow and pseudo-3D series peak fitting
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: Deep neural network peak picker supplying the cross-peak list for deconvolution
    of complex overlapping 2D spectra
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs Voigt (Gaussian–Lorentzian) line-shape fitting to resolve per-peak
    amplitude, center, and width, including jointly constrained fits of overlapping
    peaks
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:6b0fc94040c6f079e14b9b5879a794c0df0e6b4c64699a0d1884761b744078ca
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# spectral-deconvolution-integration

## Summary

Apply COLMARvista's full integration with DEEP Picker and Voigt Fitter to automatically deconvolve complex 2D/pseudo-3D NMR spectra and reconstruct them, yielding quantitative per-peak parameters (concentrations, cross-peak-specific relaxation parameters) even for significantly overlapping peaks.

## When to use

Use when 2D or pseudo-3D NMR spectra contain overlapping cross-peaks requiring quantitative per-peak parameters (amplitude, width, T1/T2 relaxation, concentration) that simple peak picking cannot resolve.

## When NOT to use

- The spectrum is well-resolved with no peak overlap — a local-maxima Simple Picker suffices without joint Voigt deconvolution.
- The goal is purely qualitative spectral viewing or NUS reconstruction (SMILE) rather than quantitative per-peak parameter extraction.
- The data are 1D spectra where the 2D cross-peak fitting machinery and pseudo-3D series fitting are not applicable.

## Inputs

- Processed or raw 2D/pseudo-3D NMR spectrum (Bruker time-domain `fid`/`ser` or frequency-domain data)
- Cross-peak coordinate/peak list from automated deconvolution (DEEP Picker output)
- Reference assignment lists (.list, .tab, .txt, .csv) for peak identification

## Outputs

- Structured CSV table of fitted per-peak Voigt parameters (amplitude, center frequency, width) — one row per picked cross-peak
- Reconstructed spectrum from the summed fitted line shapes
- Cross-peak-specific relaxation parameters (e.g., T1/T2) and compound concentrations from pseudo-3D series fitting

## How to apply

Load the processed 2D NMR spectrum (Bruker `fid`/`ser` or processed data) and a cross-peak coordinate list produced by automated deconvolution into COLMARvista. Select the Voigt Fitter module and fit each picked cross-peak with a Voigt (Gaussian–Lorentzian convolution) line shape, estimating per-peak amplitude, center frequency, and width; DEEP Picker's deep neural network supplies the peak list where local-maxima Simple Picker is insufficient. For overlapping signals, constrain the fit jointly so cross-peak-specific quantitative parameters (e.g., relaxation parameters, compound concentrations) are resolved per component. Write fitted per-peak parameters to a structured CSV table and confirm the fitted line shapes reconstruct the experimental spectrum. For pseudo-3D relaxation series (T1, T2), titration, or kinetics, run multi-plane sequential peak fitting across the series planes to extract per-residue decay parameters.

## Related tools

- **COLMARvista** (Web-based integrated suite hosting the Voigt/Gaussian 2D line-shape fitting workflow and pseudo-3D series peak fitting) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (Deep neural network peak picker supplying the cross-peak list for deconvolution of complex overlapping 2D spectra) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs Voigt (Gaussian–Lorentzian) line-shape fitting to resolve per-peak amplitude, center, and width, including jointly constrained fits of overlapping peaks)

## Examples

```
git clone https://github.com/lidawei1975/colmarvista && open colmarvista/index.html  # then use the Voigt Fitter module in-browser on the picked 2D peak list
```

## Evaluation signals

- Fitted line shapes reconstruct the experimental spectrum (automated reconstruction as described for the Voigt Fitter integration).
- Output CSV table contains exactly one row per picked cross-peak with converged fit parameters (amplitude, center frequency, width).
- Cross-peak-specific relaxation parameters are recovered even for signals affected by significant overlap with other peaks.
- For pseudo-3D T1/T2 series, sequential peak fitting across planes yields monotone, physically plausible decay behavior.
- Peak list from DEEP Picker matches expected cross-peak positions in the 2D spectrum before fitting.

## Limitations

- Local offline use requires modifying browser settings (e.g., Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari 'Disable Local File Restrictions'), which the README warns poses a security risk.
- ChemEx-based exchange analysis and related fitting run client-side via Pyodide WebWorker, so browser WebWorker/WebAssembly support is a prerequisite.
- No changelog is available in the repository.

## Evidence
