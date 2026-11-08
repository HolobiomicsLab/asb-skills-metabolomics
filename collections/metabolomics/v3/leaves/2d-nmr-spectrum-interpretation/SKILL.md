---
name: 2d-nmr-spectrum-interpretation
description: Use when raw Bruker FID data (fid, ser) or processed 2D/pseudo-3D NMR
  spectra need processing, visualization, and peak picking/deconvolution to build
  a cross-peak list with ppm coordinates.
when_to_use_negative:
- The spectrum is 1D only and no 2D/pseudo-3D contour visualization or cross-peak
  analysis is needed
- Extremely overlapped spectra requiring external dedicated deconvolution beyond DEEP
  Picker/Voigt Fitter capabilities
- A workflow requires server-side batch processing or data upload — COLMARvista is
  a static web server that accepts no uploads
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3420
tools:
- name: COLMARvista
  role: Web-based environment for processing raw Bruker FIDs, visualizing 2D/pseudo-3D
    spectra as WebGL contours, and interactive spectral analysis
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: AI-driven deep neural network for automated deconvolution of complex, overlapping
    2D NMR cross-peaks
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Voigt/Gaussian line-shape fitting for spectrum reconstruction and quantitative
    analysis of picked peaks
  repo: https://github.com/lidawei1975/colmarvista
- name: SMILE WebAssembly plugin
  role: Iterative reconstruction of non-uniformly sampled 2D, pseudo-3D, and 3D NMR
    experiments
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:8d206ccca2dfcc9029634dd343adc1a0b034b57d05fa47cf0094d5f6472b9638
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# 2d-nmr-spectrum-interpretation

## Summary

Interpret processed 2D and pseudo-3D NMR spectra using COLMARvista, a JavaScript-based, browser-side tool that visualizes WebGL contours and identifies cross-peaks via local maxima (Simple Picker), the DEEP Picker neural network, or Voigt/Gaussian deconvolution.

## When to use

Use when raw Bruker FID data (fid, ser) or processed 2D/pseudo-3D NMR spectra need processing, visualization, and peak picking/deconvolution to build a cross-peak list with ppm coordinates.

## When NOT to use

- The spectrum is 1D only and no 2D/pseudo-3D contour visualization or cross-peak analysis is needed
- Extremely overlapped spectra requiring external dedicated deconvolution beyond DEEP Picker/Voigt Fitter capabilities
- A workflow requires server-side batch processing or data upload — COLMARvista is a static web server that accepts no uploads

## Inputs

- Raw Bruker time-domain FID data (fid, ser)
- Processed (frequency-domain) 2D or pseudo-3D NMR spectra
- NUS 2D, pseudo-3D, and 3D experiment data (direct-FT or SMILE reconstruction)
- Reference peak lists (.list, .tab, .txt, .csv) for assignment transfer

## Outputs

- Peak list table with detected cross-peak positions (ppm coordinates) and intensities
- Voigt/Gaussian deconvoluted and reconstructed spectrum for quantitative analysis
- Contour plots, 1D projections/slices, and 3D WebGL isosurface visualizations
- Assigned peak lists via automated assignment transfer

## How to apply

Load raw Bruker time-domain data (fid, ser) into COLMARvista, which processes it client-side via WebAssembly with automated phase correction, apodization, zero filling, solvent suppression, and baseline correction. Inspect the frequency-domain spectrum as WebGL contours, 1D projections, or 3D surface/isosurface renderings, using synchronous zoom/pan for pseudo-3D series. For peak identification, choose between the AI-driven DEEP Picker neural network, the lightweight local-maxima Simple Picker, or Voigt/Gaussian deconvolution (Voigt Fitter) depending on peak overlap severity; DEEP Picker plus Voigt Fitter enables quantitative reconstruction for concentration determination and cross-peak specific relaxation parameters. Because everything runs in the browser via JavaScript/WebAssembly, no data leaves the user's computer; the peak list can be transferred or calibrated via automated assignment transfer with Hungarian bipartite matching against reference lists (.list, .tab, .txt, .csv).

## Related tools

- **COLMARvista** (Web-based environment for processing raw Bruker FIDs, visualizing 2D/pseudo-3D spectra as WebGL contours, and interactive spectral analysis) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (AI-driven deep neural network for automated deconvolution of complex, overlapping 2D NMR cross-peaks) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Voigt/Gaussian line-shape fitting for spectrum reconstruction and quantitative analysis of picked peaks) — https://github.com/lidawei1975/colmarvista
- **SMILE WebAssembly plugin** (Iterative reconstruction of non-uniformly sampled 2D, pseudo-3D, and 3D NMR experiments) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- Every peak in the output peak table corresponds to a local intensity maximum (Simple Picker) in the loaded processed 2D spectrum, with valid ppm coordinates
- Processed spectrum shows correct phase and baseline after automated phase correction and baseline correction of the raw fid/ser input
- Reconstructed spectrum from DEEP Picker + Voigt Fitter visually matches experimental contours, including overlapped cross-peaks
- Picked peak positions agree with reference peak lists after global calibration shift offsets and assignment-transfer distance cutoffs are applied
- No data was transmitted to a server — processing occurred entirely in-browser (static web server, no uploads)

## Limitations

- Simple Picker only detects local maxima, so heavily overlapped cross-peaks require DEEP Picker or Voigt/Gaussian deconvolution instead
- Local (file://) use requires modifying browser security settings (Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari Disable Local File Restrictions), which poses a security risk
- Browser/WebWorker/WebAssembly configuration is required for local offline use; the server does not accept uploads
- No changelog is available for the repository

## Evidence
