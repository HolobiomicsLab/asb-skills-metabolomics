---
name: nmr-fourier-transform-processing
description: 'Use when raw Bruker time-domain 2D NMR FID data (`fid` or `ser`) must
  be converted to a phased 2D spectrum via the automated chain: zero-filling, apodization,
  water suppression, Fourier transformation, phase correction.'
when_to_use_negative:
- The data are already frequency-domain (processed) spectra that only need viewing,
  peak picking, or 2D line-shape fitting rather than time-domain reconstruction.
- The experiment is NUS and the goal is iterative SMILE reconstruction of pseudo-3D/3D
  data, which is a different branch of COLMARvista functionality than the conventional
  autonomous chain.
- Interactive, user-guided manual processing is required — this chain is defined by
  COLMARvista's fully autonomous (no user intervention) behavior.
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Reference implementation of the fully autonomous processing chain (zero-filling,
    apodization, water suppression, Fourier transformation, phase correction) whose
    behavior the standalone scripts reconstruct
  repo: https://github.com/lidawei1975/colmarvista
- name: WebAssembly
  role: Execution substrate through which COLMARvista processes raw Bruker time-domain
    FID data client-side; a standalone reimplementation replaces this with a scriptable
    processing environment
- name: SMILE WebAssembly plugin
  role: Iterative reconstruction backend used instead of conventional FT for NUS experiments
    within the same processing environment
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:38d415e1d94938114bc534dd4e93922b359f478c3a9f9e17401d9ebfa8105974
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# nmr-fourier-transform-processing

## Summary

Reconstructs the autonomous 2D NMR spectral processing chain used by COLMARvista — zero-filling → apodization → water (solvent) suppression → Fourier transformation → automated phase correction — as standalone inspectable steps applied to raw Bruker time-domain FID data.

## When to use

Use when raw Bruker time-domain 2D NMR FID data (`fid` or `ser`) must be converted to a phased 2D spectrum via the automated chain: zero-filling, apodization, water suppression, Fourier transformation, phase correction.

## When NOT to use

- The data are already frequency-domain (processed) spectra that only need viewing, peak picking, or 2D line-shape fitting rather than time-domain reconstruction.
- The experiment is NUS and the goal is iterative SMILE reconstruction of pseudo-3D/3D data, which is a different branch of COLMARvista functionality than the conventional autonomous chain.
- Interactive, user-guided manual processing is required — this chain is defined by COLMARvista's fully autonomous (no user intervention) behavior.

## Inputs

- Raw Bruker 2D time-domain FID data file (`fid` or `ser`)
- Optionally, NUS experiment data for 2D / pseudo-3D / 3D direct-FT or SMILE reconstruction

## Outputs

- Final phased 2D spectrum (spectrum_phased.dat), loadable for viewing/analysis
- Intermediate files: fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat

## How to apply

Load the raw Bruker time-domain FID (`fid` or `ser`) file — COLMARvista does this client-side in the browser via WebAssembly, so a standalone reimplementation should process the same time-domain input. Apply the COLMARvista autonomous chain in its published order: zero-filling to extend the time-domain data, apodization (window function) to the zero-filled FID, water/solvent suppression to the apodized FID, then Fourier transformation along both dimensions to obtain the 2D spectrum. Finish with automated phase correction (README notes COLMARvista also applies baseline correction at this stage, which can be included as an optional extra step). Persist each intermediate as a separate inspectable file (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat, spectrum_phased.dat) so the chain can be validated step by step. For non-uniformly sampled experiments, substitute the conventional FT with fast direct-FT or iterative SMILE reconstruction rather than the standard pipeline.

## Related tools

- **COLMARvista** (Reference implementation of the fully autonomous processing chain (zero-filling, apodization, water suppression, Fourier transformation, phase correction) whose behavior the standalone scripts reconstruct) — https://github.com/lidawei1975/colmarvista
- **WebAssembly** (Execution substrate through which COLMARvista processes raw Bruker time-domain FID data client-side; a standalone reimplementation replaces this with a scriptable processing environment)
- **SMILE WebAssembly plugin** (Iterative reconstruction backend used instead of conventional FT for NUS experiments within the same processing environment) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- Each intermediate file (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat) exists and is non-empty.
- The final phased 2D spectrum file (spectrum_phased.dat) is produced and loadable by a 2D NMR viewer.
- The step order matches COLMARvista's autonomous chain exactly: zero-filling → apodization → water suppression → Fourier transformation → phase correction.
- Processing runs end-to-end with no user-set phase parameters, mirroring COLMARvista's 'fully autonomous processing of spectra'.
- For NUS inputs, the result is produced via direct-FT or SMILE reconstruction rather than the conventional FT path.

## Limitations

- Running locally requires modified browser settings for WebWorker/WebAssembly (Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari Develop > Disable Local File Restrictions), which the README flags as a security risk.
- The proprietary SMILE WebAssembly plugin is only usable under a GPL-V3 linking exception with the original author's permission.
- README describes the client-side processing as also including baseline correction, which is outside the five-step abstract-level chain and may be an optional or version-dependent step.
- No changelog found (per extraction), so version drift in the autonomous chain cannot be tracked from the repository.

## Evidence
