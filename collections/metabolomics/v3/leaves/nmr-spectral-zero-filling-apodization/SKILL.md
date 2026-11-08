---
name: nmr-spectral-zero-filling-apodization
description: Use when starting from raw Bruker time-domain FID files (fid or ser)
  and an unprocessed 2D or pseudo-3D NMR spectrum is needed; triggers include 'free
  induction decay' input and no phased spectrum yet.
when_to_use_negative:
- Input is already a processed, phased 2D spectrum rather than raw time-domain fid/ser
  data
- Input is non-uniformly sampled (NUS) data requiring direct-FT or iterative SMILE
  reconstruction instead of the conventional processing chain
- The goal is peak deconvolution or line-shape fitting (DEEP Picker / Voigt Fitter)
  rather than producing a phased spectrum
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: JavaScript/WebAssembly web application that performs the autonomous zero-filling,
    apodization, water suppression, Fourier transformation, and phase correction chain
    client-side
  repo: https://github.com/lidawei1975/colmarvista
- name: WebAssembly
  role: Runtime layer (with the SMILE plugin) that executes raw Bruker FID processing
    directly in the browser
- name: SMILE WebAssembly plugin
  role: Proprietary plugin linked into COLMARvista used for spectral processing/reconstruction
    functionality
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:1bc57ff62e34176893113839208db4b01e718b6ac4582d81c7453bea627aff2e
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - JavaScript
  - SMILE WebAssembly plugin
  - WebAssembly
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nmr-spectral-zero-filling-apodization

## Summary

A skill for autonomously processing raw Bruker 2D time-domain NMR FID data (fid/ser) through the COLMARvista chain of zero-filling, apodization, water suppression, Fourier transformation, and phase correction, producing a phased 2D spectrum without user intervention.

## When to use

Use when starting from raw Bruker time-domain FID files (fid or ser) and an unprocessed 2D or pseudo-3D NMR spectrum is needed; triggers include 'free induction decay' input and no phased spectrum yet.

## When NOT to use

- Input is already a processed, phased 2D spectrum rather than raw time-domain fid/ser data
- Input is non-uniformly sampled (NUS) data requiring direct-FT or iterative SMILE reconstruction instead of the conventional processing chain
- The goal is peak deconvolution or line-shape fitting (DEEP Picker / Voigt Fitter) rather than producing a phased spectrum

## Inputs

- Raw Bruker 2D time-domain FID data file (fid)
- Raw Bruker 2D time-domain serialized data file (ser)

## Outputs

- Zero-filled time-domain data (fid_zero_filled.dat)
- Apodized time-domain data (fid_apodized.dat)
- Solvent-suppressed time-domain data (fid_solvent_suppressed.dat)
- Fourier-transformed 2D spectrum (spectrum_ft.dat)
- Automatically phase-corrected 2D spectrum (spectrum_phased.dat)

## How to apply

Load the raw Bruker 2D time-domain data (fid or ser) into the processing environment (COLMARvista processes it client-side via its WebAssembly/SMILE pipeline). Apply the autonomous chain in order: zero-filling to extend the time-domain data, apodization (window function) to the zero-filled FID, water/solvent suppression to the apodized FID, Fourier transformation along both dimensions, then automated phase correction; baseline correction is also part of the browser-side automated chain. The rationale for the order is that windowing and solvent filtering must operate on the time-domain signal before FT, and phase correction only makes sense on the transformed spectrum. For reproducibility, save each intermediate result as a separate inspectable file (e.g. fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat, spectrum_phased.dat) so the chain can be validated stepwise.

## Related tools

- **COLMARvista** (JavaScript/WebAssembly web application that performs the autonomous zero-filling, apodization, water suppression, Fourier transformation, and phase correction chain client-side) — https://github.com/lidawei1975/colmarvista
- **WebAssembly** (Runtime layer (with the SMILE plugin) that executes raw Bruker FID processing directly in the browser)
- **SMILE WebAssembly plugin** (Proprietary plugin linked into COLMARvista used for spectral processing/reconstruction functionality)

## Evaluation signals

- Each intermediate file (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat, spectrum_phased.dat) exists and is non-empty
- The final phased 2D spectrum file is produced and loadable in the viewing tool
- The five processing steps occurred in the documented order: zero-filling → apodization → water suppression → Fourier transformation → phase correction
- The phased spectrum shows correctly phased peaks (no dispersive distortion), consistent with 'automated phase correction' applied without user intervention

## Limitations

- Running COLMARvista locally from index.html requires manual browser configuration (Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, or Safari local-file restrictions), which poses a security risk
- The proprietary SMILE WebAssembly plugin is linked via a linking exception rather than being fully GPL-licensed
- No changelog is available for tracking processing-chain changes

## Evidence
