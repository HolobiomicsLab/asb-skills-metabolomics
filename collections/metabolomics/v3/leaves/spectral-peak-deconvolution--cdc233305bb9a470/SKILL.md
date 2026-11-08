---
name: spectral-peak-deconvolution
description: Use when a processed 2D or pseudo-3D NMR spectrum contains cross-peaks
  significantly overlapped with neighboring signals and an accurate per-peak list
  (positions, intensities) is needed for concentration or relaxation-parameter analysis.
when_to_use_negative:
- The spectrum is 1D NMR or raw unprocessed time-domain FID data — first run processing/FT
  (the skill targets 2D and pseudo-3D spectra already processed)
- Cross-peaks are well separated and a local-maxima Simple Picker alone suffices —
  neural-network deconvolution is unnecessary overhead
- The task is CEST dip pre-analysis or NUS reconstruction (SMILE) rather than resolving
  overlapped cross-peaks — these are separate COLMARvista functions
edam_operation: http://edamontology.org/operation_0327
tools:
- name: DEEP Picker
  role: Deep neural network peak picker that detects candidate cross-peak positions
    and drives automated deconvolution of overlapped 2D spectra
- name: Voigt Fitter
  role: Performs Voigt/Gaussian 2D line-shape fitting of the resolved components so
    the spectrum can be reconstructed and quantified
- name: COLMARvista
  role: Web-based environment that integrates DEEP Picker and Voigt Fitter for interactive
    peak picking, deconvolution, and pseudo-3D series fitting
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:8bdfb7bdb15fffe982e02c776917dee4157468e7cf543c376c2524940d7f3833
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
  edam_operation: http://edamontology.org/operation_0327
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spectral-peak-deconvolution

## Summary

Automated deconvolution of overlapping 2D NMR cross-peaks using the deep-neural-network DEEP Picker combined with Voigt/Gaussian line-shape fitting in COLMARvista, resolving overlapped signals into individually quantifiable components.

## When to use

Use when a processed 2D or pseudo-3D NMR spectrum contains cross-peaks significantly overlapped with neighboring signals and an accurate per-peak list (positions, intensities) is needed for concentration or relaxation-parameter analysis.

## When NOT to use

- The spectrum is 1D NMR or raw unprocessed time-domain FID data — first run processing/FT (the skill targets 2D and pseudo-3D spectra already processed)
- Cross-peaks are well separated and a local-maxima Simple Picker alone suffices — neural-network deconvolution is unnecessary overhead
- The task is CEST dip pre-analysis or NUS reconstruction (SMILE) rather than resolving overlapped cross-peaks — these are separate COLMARvista functions

## Inputs

- Processed 2D NMR spectrum file (e.g., Fourier-transformed Bruker data)
- Pseudo-3D NMR series planes (relaxation T1/T2, titration, or kinetics)
- Optional local-maxima picker peak list for cross-checking

## Outputs

- Resolved per-spectrum peak list (deconvoluted cross-peak positions)
- Voigt/Gaussian-fitted component line shapes and reconstructed spectrum
- Quantitative per-peak parameters (intensities/concentrations, cross-peak relaxation parameters)

## How to apply

Load a processed 2D spectrum (e.g., post-FT Bruker data with phase correction, apodization, zero filling, and solvent suppression already applied) into COLMARvista and feed it to the DEEP Picker deep neural network to generate an initial list of candidate cross-peak positions. Run the automated deconvolution so overlapping cross-peaks are resolved into individual components, then fit each component with Voigt or Gaussian line shapes (Voigt Fitter) to reconstruct the experimental spectrum quantitatively. When residual ambiguities remain, compare or combine the DEEP Picker output with the local-maxima Simple Picker output to decide the final peak list. The deconvoluted peak list then supports downstream quantitative analyses such as compound concentration determination and extraction of cross-peak-specific relaxation parameters (T1, T2) across pseudo-3D series.

## Related tools

- **DEEP Picker** (Deep neural network peak picker that detects candidate cross-peak positions and drives automated deconvolution of overlapped 2D spectra)
- **Voigt Fitter** (Performs Voigt/Gaussian 2D line-shape fitting of the resolved components so the spectrum can be reconstructed and quantified)
- **COLMARvista** (Web-based environment that integrates DEEP Picker and Voigt Fitter for interactive peak picking, deconvolution, and pseudo-3D series fitting) — https://github.com/lidawei1975/colmarvista

## Examples

```
open https://lidawei1975.github.io/colmarvista/  # load processed 2D spectrum, Peak Picking → DEEP Picker, then Voigt/Gaussian deconvolution
```

## Evaluation signals

- Reconstruction check: re-synthesizing the spectrum from the fitted Voigt components should reproduce the experimental spectrum, including under overlapped regions
- Overlapped cross-peaks that a local-maxima picker resolves as a single maximum should be split into ≥2 components by the DEEP Picker deconvolution
- Peak positions in the final deconvoluted list should be consistent with the initial DNN-generated candidate list, with ambiguities reconciled against Simple Picker output
- Downstream quantitative outputs (compound concentrations, cross-peak-specific T1/T2 relaxation parameters) should be extractable from the resolved components
- The final deconvoluted peak-list file should be written as the workflow's output artifact for each spectrum

## Limitations

- Residual ambiguities in heavily overlapped regions may require optional comparison/combination with the local-maxima picker output
- Local (offline) use requires modifying browser security settings (e.g., Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari 'Disable Local File Restrictions'), which the README flags as a security risk
- No changelog was found in the repository documentation

## Evidence
