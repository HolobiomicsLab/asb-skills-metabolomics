---
name: nmr-spectral-peak-picking
description: Use when you have a processed, frequency-domain 2D NMR spectrum (after
  Fourier transformation, phasing, baseline correction) and need a peak list of cross-peak
  positions and intensities via local maxima detection.
when_to_use_negative:
- Spectrum is still a raw time-domain FID — process it first (Fourier transform, phase
  correction, baseline correction) before peak picking
- Peaks are heavily overlapped and the goal is quantitative deconvolution (concentration
  determination, cross-peak specific relaxation parameters) — use DEEP Picker with
  Voigt Fitter instead of local maxima detection
- The task is CEST dip pre-analysis or exchange-active residue filtering ('Show ≥2
  Peaks Only'), which are separate COLMARvista workflows
edam_operation: null
tools:
- name: COLMARvista
  role: Web-based environment hosting the Simple Picker local-maxima peak detection
    on processed 2D NMR spectra
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: Alternative AI-driven neural-network picker for automated deconvolution of
    complex 2D spectra when local maxima are insufficient
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Voigt/Gaussian deconvolution companion for quantitative reconstruction of
    picked peaks
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:550d2a9f748b574d4e39150515c4a851119c7ca67d54a42ff854c693b9577c9a
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
yes_rate: 0.5
n_claims: 4
---

# nmr-spectral-peak-picking

## Summary

Extracts cross-peak positions and intensities from a processed 2D NMR spectrum by scanning the spectral matrix for local intensity maxima, using COLMARvista's lightweight Simple Picker alongside its AI-driven DEEP Picker and Voigt/Gaussian deconvolution options.

## When to use

Use when you have a processed, frequency-domain 2D NMR spectrum (after Fourier transformation, phasing, baseline correction) and need a peak list of cross-peak positions and intensities via local maxima detection.

## When NOT to use

- Spectrum is still a raw time-domain FID — process it first (Fourier transform, phase correction, baseline correction) before peak picking
- Peaks are heavily overlapped and the goal is quantitative deconvolution (concentration determination, cross-peak specific relaxation parameters) — use DEEP Picker with Voigt Fitter instead of local maxima detection
- The task is CEST dip pre-analysis or exchange-active residue filtering ('Show ≥2 Peaks Only'), which are separate COLMARvista workflows

## Inputs

- Processed 2D NMR spectrum (frequency-domain data after Fourier transformation, phasing, and baseline correction)
- Raw Bruker time-domain FID data (`fid`, `ser`) if in-browser processing precedes peak picking

## Outputs

- Peak list table of detected local maxima with ppm coordinates and intensities

## How to apply

Load the processed 2D spectrum into COLMARvista (raw Bruker `fid`/`ser` files can be processed client-side via WebAssembly with automated phase correction, apodization, zero filling, solvent suppression, and baseline correction first). Invoke the Simple Picker, which scans the 2D spectral matrix for local intensity maxima corresponding to cross-peak positions, rather than the AI-driven DEEP Picker neural network or Voigt/Gaussian deconvolution, which are intended for heavier overlapping-peak analysis. Compile the detected peak positions and intensities into a peak list table with ppm coordinates. The decision point between pickers is analysis goal: Simple Picker for fast local-maxima detection on clean spectra; DEEP Picker/Voigt Fitter when automated deconvolution and reconstruction are needed for quantitative analysis such as compound concentration or cross-peak-specific relaxation parameters.

## Related tools

- **COLMARvista** (Web-based environment hosting the Simple Picker local-maxima peak detection on processed 2D NMR spectra) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (Alternative AI-driven neural-network picker for automated deconvolution of complex 2D spectra when local maxima are insufficient) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Voigt/Gaussian deconvolution companion for quantitative reconstruction of picked peaks) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- The output peak table lists all detected local maxima with their ppm coordinates
- Every detected peak corresponds to a local intensity maximum in the loaded spectrum
- Detected cross-peak positions visually coincide with contour maxima in the WebGL 2D spectrum view
- Peak count is consistent with the spectrum's complexity; if peaks are missed due to overlap, escalate to DEEP Picker/Voigt Fitter and compare peak lists

## Limitations

- Simple Picker is a lightweight local-maxima detector; it does not deconvolve overlapping peaks, so quantitative concentration or relaxation analysis requires DEEP Picker and Voigt Fitter integration
- When running COLMARvista locally from index.html, WebWorker/WebAssembly support requires manual browser configuration (e.g., --allow-file-access-from-files in Chrome), which the README flags as a security risk

## Evidence
