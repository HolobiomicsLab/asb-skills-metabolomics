---
name: nmr-automated-phase-correction
description: Use when raw Bruker time-domain FID data (`fid` or `ser`) must be turned
  into a correctly phased 2D spectrum without manual phasing, e.g. batch or autonomous
  processing of many spectra.
when_to_use_negative:
- Input is already a frequency-domain 2D spectrum that is properly phased — only downstream
  viewing/peak picking (DEEP Picker, Voigt Fitter) is needed
- The task is NUS reconstruction (direct-FT or iterative SMILE) rather than conventional
  FID processing
- The user needs manual, interactive phase adjustment of individual spectra rather
  than autonomous processing
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Web-based (JavaScript) processing pipeline that applies automated phase correction
    (along with apodization, zero filling, solvent suppression, and baseline correction)
    to raw Bruker FID data
  repo: https://github.com/lidawei1975/colmarvista
- name: WebAssembly
  role: Client-side execution engine (with the proprietary SMILE WebAssembly plugin)
    that lets the phase-correction chain run directly in the browser on local FID
    files
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:6a4f7c97592b6279f634d5e3569f9640d8b5eed52cee228f76f16bbda42fdb71
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nmr-automated-phase-correction

## Summary

Automated phase correction of 2D NMR spectra as part of COLMARvista's fully autonomous processing chain (zero-filling → apodization → water suppression → Fourier transformation → phase correction), executed client-side in WebAssembly on raw Bruker FID data.

## When to use

Use when raw Bruker time-domain FID data (`fid` or `ser`) must be turned into a correctly phased 2D spectrum without manual phasing, e.g. batch or autonomous processing of many spectra.

## When NOT to use

- Input is already a frequency-domain 2D spectrum that is properly phased — only downstream viewing/peak picking (DEEP Picker, Voigt Fitter) is needed
- The task is NUS reconstruction (direct-FT or iterative SMILE) rather than conventional FID processing
- The user needs manual, interactive phase adjustment of individual spectra rather than autonomous processing

## Inputs

- Raw Bruker time-domain FID data files (`fid` or `ser`)

## Outputs

- Phased 2D NMR spectrum (spectrum_phased.dat)
- Intermediate processing files (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat)

## How to apply

Load the raw Bruker 2D time-domain FID (`fid` or `ser`) into COLMARvista, which processes it client-side via its SMILE WebAssembly plugin so all data stays on the user's computer. Run the autonomous chain in order: zero-filling to extend the time-domain data, apodization (window function), water/solvent suppression, Fourier transformation along both dimensions, then automated phase correction of the transformed spectrum. Save each intermediate as a separate inspectable file (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat, spectrum_phased.dat) so the phasing step can be audited independently. The rationale is that COLMARvista applies this chain automatically — no user-adjusted zero- and first-order phase parameters are required — making it suitable for pseudo-3D series and repetitive 2D datasets.

## Related tools

- **COLMARvista** (Web-based (JavaScript) processing pipeline that applies automated phase correction (along with apodization, zero filling, solvent suppression, and baseline correction) to raw Bruker FID data) — https://github.com/lidawei1975/colmarvista
- **WebAssembly** (Client-side execution engine (with the proprietary SMILE WebAssembly plugin) that lets the phase-correction chain run directly in the browser on local FID files) — https://github.com/lidawei1975/colmarvista

## Examples

```
npm start  # serve COLMARvista at http://127.0.0.1:8080, then open index.html and load Bruker fid/ser for autonomous processing
```

## Evaluation signals

- The final phased 2D spectrum file (spectrum_phased.dat) exists, is non-empty, and is loadable in COLMARvista
- Every intermediate file in the chain (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat) exists and is non-empty, confirming the phase-correction input was correctly produced
- The phased spectrum shows absorptive (positive) cross-peaks rather than dispersive/distorted lineshapes, consistent with successful automated phasing
- The processed spectrum is consistent with COLMARvista's documented autonomous chain output for the same `fid`/`ser` input (reproducibility check)

## Limitations

- Running COLMARvista locally from index.html requires modifying browser settings for WebWorker/WebAssembly (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy = false), which the README warns poses a security risk
- Phase correction runs after the full upstream chain (zero-filling, apodization, solvent suppression, FT); errors in earlier steps propagate into the phased result
- The processing is autonomous — users do not tune phase parameters, so unusual spectra requiring manual phase refinement may need a different tool

## Evidence
