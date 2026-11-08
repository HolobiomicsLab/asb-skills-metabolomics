---
name: nmr-time-domain-fid-processing
description: Use when raw Bruker time-domain FID files (`fid` or `ser`) must be turned
  into inspectable, phased 2D spectra and no manually tuned processing parameters
  are available or desired.
when_to_use_negative:
- Input is already frequency-domain / processed 2D data that only needs viewing or
  peak picking — use the viewing and DEEP Picker/Voigt Fitter functions instead
- Input is non-uniformly sampled (NUS) data requiring direct-FT or iterative SMILE
  reconstruction — that is a separate reconstruction workflow, not this autonomous
  FID chain
- The goal is quantitative deconvolution (concentrations, cross-peak relaxation parameters)
  — this skill only produces the phased spectrum that feeds DEEP Picker and Voigt
  Fitter
edam_operation: http://edamontology.org/operation_0325
tools:
- name: COLMARvista
  role: Runs the autonomous zero-filling → apodization → water suppression → Fourier
    transformation → phase correction chain on raw Bruker FID data
  repo: https://github.com/lidawei1975/colmarvista
- name: WebAssembly
  role: Client-side execution engine that processes the `fid`/`ser` time-domain data
    directly in the browser with no uploads
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:78acf629a053f85b43c193e1c2ebcc9e2cf92c7d17e9ff372ed3f29df8526102
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - JavaScript
  - WebAssembly
  edam_operation: http://edamontology.org/operation_0325
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# nmr-time-domain-fid-processing

## Summary

Autonomous conversion of raw Bruker time-domain FID data (`fid`, `ser`) into phased 2D NMR spectra by executing COLMARvista's client-side WebAssembly chain: zero-filling → apodization → solvent/water suppression → Fourier transformation → automated phase correction, without user intervention.

## When to use

Use when raw Bruker time-domain FID files (`fid` or `ser`) must be turned into inspectable, phased 2D spectra and no manually tuned processing parameters are available or desired.

## When NOT to use

- Input is already frequency-domain / processed 2D data that only needs viewing or peak picking — use the viewing and DEEP Picker/Voigt Fitter functions instead
- Input is non-uniformly sampled (NUS) data requiring direct-FT or iterative SMILE reconstruction — that is a separate reconstruction workflow, not this autonomous FID chain
- The goal is quantitative deconvolution (concentrations, cross-peak relaxation parameters) — this skill only produces the phased spectrum that feeds DEEP Picker and Voigt Fitter

## Inputs

- Raw Bruker time-domain FID data files (`fid` or `ser`)
- 2D or pseudo-3D NMR experiment FID series

## Outputs

- Zero-filled time-domain intermediate (e.g. fid_zero_filled.dat)
- Apodized FID intermediate (e.g. fid_apodized.dat)
- Solvent-suppressed FID intermediate (e.g. fid_solvent_suppressed.dat)
- Fourier-transformed 2D spectrum (e.g. spectrum_ft.dat)
- Phased 2D spectrum (e.g. spectrum_phased.dat), loadable for visualization and downstream peak picking

## How to apply

Load the raw Bruker time-domain `fid` or `ser` file directly in the browser (COLMARvista processes it client-side via WebAssembly, so data never leaves the machine). Apply the fixed autonomous chain in order: zero-filling to extend the time-domain data, apodization (window function) on the zero-filled FID, water/solvent suppression on the apodized FID, Fourier transformation along both dimensions, then automated phase correction (methods also list baseline correction as part of the automated browser-side processing). Save each intermediate as a separate inspectable file (e.g. fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat, spectrum_phased.dat) so the chain can be audited step by step. The rationale for full automation is that these steps are parameter-independent enough that COLMARvista performs them 'fully autonomously... including zero-filling, apodization, water suppression, Fourier transformation, and phase correction', removing a major manual bottleneck before downstream peak picking (DEEP Picker/Simple Picker) or Voigt/Gaussian deconvolution.

## Related tools

- **COLMARvista** (Runs the autonomous zero-filling → apodization → water suppression → Fourier transformation → phase correction chain on raw Bruker FID data) — https://github.com/lidawei1975/colmarvista
- **WebAssembly** (Client-side execution engine that processes the `fid`/`ser` time-domain data directly in the browser with no uploads)

## Evaluation signals

- Each intermediate file (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat) exists and is non-empty
- The final phased 2D spectrum file (spectrum_phased.dat) is produced and loadable by COLMARvista's WebGL contour viewer
- The spectrum shows absorptive (properly phased) cross-peak lineshapes after the automated phase correction step
- Re-running the chain on the same `fid`/`ser` input reproduces identical intermediate and final files (deterministic, user-parameter-free pipeline)
- The phased output can be fed directly to DEEP Picker / Voigt Fitter for deconvolution without an extra manual processing pass

## Limitations

- Local/offline use requires modifying browser security settings (Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari Disable Local File Restrictions), which the README flags as a security risk
- Processing depends on WebWorker/WebAssembly support and the proprietary SMILE WebAssembly plugin, which is only usable under a GPL-V3 linking exception 'used with permission from its original author'
- No changelog is available, so version-to-version behavior of the automated parameters cannot be tracked from the repository

## Evidence
