---
name: deep-neural-network-peak-picking
description: Use when a processed 2D or pseudo-3D NMR spectrum contains significantly
  overlapped cross-peaks that defeat local-maxima picking, and quantitative per-peak
  intensities, concentrations, or relaxation parameters are needed.
when_to_use_negative:
- When cross-peaks are well separated — the local-maxima Simple Picker is the simpler
  appropriate choice
- When the input is a raw unprocessed time-domain FID (`fid`/`ser`) that still requires
  phase correction, apodization, zero filling, and solvent suppression before peak
  picking
- When the task is pure viewing/visualization of contours without peak extraction
  or line-shape fitting
edam_operation: http://edamontology.org/operation_3646
tools:
- name: DEEP Picker
  role: Deep neural network that performs the peak picking and automated deconvolution
    of overlapping 2D cross-peaks
  repo: https://github.com/lidawei1975/colmarvista
- name: COLMARvista
  role: Web-based (JavaScript/WebAssembly) environment that fully integrates DEEP
    Picker, provides the processed spectrum input, and hosts the combined picker workflow
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs the Voigt/Gaussian 2D line-shape deconvolution and reconstruction
    that consumes the DEEP Picker peak list for quantitative analysis
  repo: https://github.com/lidawei1975/colmarvista
- name: Simple Picker
  role: Local-maxima picker whose output is compared/combined with the DEEP Picker
    list to resolve residual ambiguities
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:bdc53e432975a5cba78b0bcf66ef74c9d3a35fa0877df78e3a80e60b782b42ef
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - DEEP Picker
  - Simple Picker
  - Voigt Fitter
  edam_operation: http://edamontology.org/operation_3646
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# deep-neural-network-peak-picking

## Summary

AI-driven 2D NMR peak picking in which the DEEP Picker deep neural network detects cross-peak positions and resolves overlapping cross-peaks in a processed 2D spectrum, feeding Voigt/Gaussian deconvolution for quantitative analysis.

## When to use

Use when a processed 2D or pseudo-3D NMR spectrum contains significantly overlapped cross-peaks that defeat local-maxima picking, and quantitative per-peak intensities, concentrations, or relaxation parameters are needed.

## When NOT to use

- When cross-peaks are well separated — the local-maxima Simple Picker is the simpler appropriate choice
- When the input is a raw unprocessed time-domain FID (`fid`/`ser`) that still requires phase correction, apodization, zero filling, and solvent suppression before peak picking
- When the task is pure viewing/visualization of contours without peak extraction or line-shape fitting

## Inputs

- Processed 2D or pseudo-3D NMR spectrum file (e.g., Bruker `ser`/processed 2D data)
- Optionally, a local-maxima Simple Picker peak list for cross-checking

## Outputs

- Per-spectrum peak list of candidate cross-peak positions (final deconvoluted output file)
- Deconvoluted per-component peak parameters suitable for Voigt/Gaussian line-shape fitting and quantitative analysis

## How to apply

Load a processed 2D spectrum file into COLMARvista and feed it to the DEEP Picker deep neural network to generate an initial peak list of candidate cross-peak positions. Optionally refine the list by comparing or combining it with the local-maxima Simple Picker output to handle residual ambiguities. Run automated deconvolution with Voigt/Gaussian line-shape fitting (Voigt Fitter) so overlapping cross-peaks are resolved into individual components and the experimental spectrum is reconstructed. The resulting per-spectrum peak list is written as the final deconvoluted output file and can be used downstream for compound concentration determination or extraction of cross-peak specific relaxation parameters in pseudo-3D (T1/T2, titration, kinetics) series.

## Related tools

- **DEEP Picker** (Deep neural network that performs the peak picking and automated deconvolution of overlapping 2D cross-peaks) — https://github.com/lidawei1975/colmarvista
- **COLMARvista** (Web-based (JavaScript/WebAssembly) environment that fully integrates DEEP Picker, provides the processed spectrum input, and hosts the combined picker workflow) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs the Voigt/Gaussian 2D line-shape deconvolution and reconstruction that consumes the DEEP Picker peak list for quantitative analysis) — https://github.com/lidawei1975/colmarvista
- **Simple Picker** (Local-maxima picker whose output is compared/combined with the DEEP Picker list to resolve residual ambiguities) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- The generated peak list contains candidate cross-peak positions for peaks that are significantly overlapped with other peaks, not only isolated maxima
- Reconstructing the spectrum from the resolved Voigt/Gaussian components reproduces the experimental spectrum
- DEEP Picker detections agree with (or sensibly extend) the Simple Picker local-maxima list after the optional comparison step
- Downstream quantities — compound concentrations or cross-peak specific relaxation parameters from pseudo-3D series — are consistent across the fitted components

## Limitations

- Residual ambiguities may remain after neural-network picking, requiring comparison/combination with Simple Picker local-maxima output
- DEEP Picker targets 2D NMR spectra; raw FID data must first be processed (automated phase correction, apodization, zero filling, solvent suppression, baseline correction) in COLMARvista before picking
- Local (file://) use of COLMARvista requires browser configuration (e.g., Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari Disable Local File Restrictions) for WebWorker/WebAssembly, which poses a security risk

## Evidence
