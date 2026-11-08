---
name: voigt-peak-shape-modeling
description: Use when 2D or pseudo-3D NMR cross-peaks overlap and concentrations,
  or cross-peak specific relaxation parameters (T1/T2, titration, DOSY), must be extracted
  from deconvoluted components rather than raw peak heights.
when_to_use_negative:
- Cross-peaks are well isolated, so simple local-maxima Simple Picker peak heights
  suffice without line-shape deconvolution
- The data are 1D spectra or non-NMR data where 2D cross-peak Voigt models do not
  apply
- The goal is NUS reconstruction (SMILE) of undersampled data rather than deconvolution
  of already-processed spectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3420
tools:
- name: COLMARvista
  role: Web-based platform that hosts interactive 2D visualization and drives the
    deconvolution/fitting workflow on loaded spectra
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: Deep-neural-network peak picker that generates the initial candidate cross-peak
    list fed to the Voigt/Gaussian fitting
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs the automated Voigt line-shape deconvolution and spectrum reconstruction
    for quantitative analysis
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:89e7a080e58ce87ae4029c6202f07d26daf8301b866bab58a0fe8b80cba294a3
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# voigt-peak-shape-modeling

## Summary

Model overlapping 2D NMR cross-peaks with Voigt (and Gaussian) line shapes integrated into COLMARvista, using DEEP Picker peak lists as seeds, to deconvolute and reconstruct experimental spectra for quantitative analysis.

## When to use

Use when 2D or pseudo-3D NMR cross-peaks overlap and concentrations, or cross-peak specific relaxation parameters (T1/T2, titration, DOSY), must be extracted from deconvoluted components rather than raw peak heights.

## When NOT to use

- Cross-peaks are well isolated, so simple local-maxima Simple Picker peak heights suffice without line-shape deconvolution
- The data are 1D spectra or non-NMR data where 2D cross-peak Voigt models do not apply
- The goal is NUS reconstruction (SMILE) of undersampled data rather than deconvolution of already-processed spectra

## Inputs

- Processed 2D NMR spectrum (e.g. Bruker-processed 2D data loaded into COLMARvista)
- Raw Bruker time-domain FID data (fid, ser) processed client-side with apodization, zero filling, phase correction
- DEEP Picker initial peak list of candidate cross-peak positions
- Pseudo-3D series planes (T1, T2, titration, kinetics) or DOSY gradient series

## Outputs

- Resolved per-spectrum peak list with deconvoluted Voigt/Gaussian components
- Reconstructed experimental spectrum from fitted components
- Quantitative parameters: compound concentrations and cross-peak specific relaxation parameters
- Multi-plane sequential peak-fitting results across pseudo-3D series

## How to apply

Load a processed 2D spectrum into COLMARvista and generate candidate cross-peak positions with the AI-driven DEEP Picker neural network, optionally cross-checking with the local-maxima Simple Picker to resolve residual ambiguities. Fit each candidate position with a Voigt (or Gaussian) line-shape model, combining/merging nearby candidates before refinement, and reconstruct the spectrum from the fitted components. Iterate pseudo-3D series fitting plane-by-plane across relaxation (T1/T2), titration, and kinetics series, or pulsed field gradient DOSY decay fits, carrying the peak list across planes. Judge success by visual overlay of reconstructed vs. experimental contours, agreement of fitted positions with the DEEP Picker list, and quantitative consistency of derived parameters (concentrations, relaxation rates) across replicates.

## Related tools

- **COLMARvista** (Web-based platform that hosts interactive 2D visualization and drives the deconvolution/fitting workflow on loaded spectra) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (Deep-neural-network peak picker that generates the initial candidate cross-peak list fed to the Voigt/Gaussian fitting) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs the automated Voigt line-shape deconvolution and spectrum reconstruction for quantitative analysis) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- Reconstructed spectrum from fitted Voigt components visually matches the experimental 2D contours in COLMARvista's WebGL viewer
- Resolved per-spectrum peak list reconciles with DEEP Picker output after combining with Simple Picker local-maxima results
- Extracted cross-peak specific relaxation parameters (T1, T2) and compound concentrations are self-consistent across pseudo-3D planes
- Overlapped cross-peaks are resolved into individually quantifiable components rather than merged single maxima
- Fitted peak positions fall within expected ^1H / heteronucleus diamond overlay after automated assignment transfer

## Limitations

- Residual ambiguities remain where peaks heavily overlap; combining DEEP Picker with local-maxima picker output is optional and may not fully resolve them
- Local offline use requires browser configuration for WebWorker/WebAssembly (e.g. --allow-file-access-from-files in Chrome), which the README flags as a security risk
- Accurate fitting depends on quality of the initial DEEP Picker candidate list and prior spectral processing (phase correction, apodization, baseline)

## Evidence

- [other] Workflow card: deconvolution into individual components: [evidence span withheld — anchor status: not_found]
