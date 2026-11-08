---
name: nmr-spectral-processing
description: Use when raw Bruker fid/ser time-domain NMR FID data needs processing
  into a phased, baseline-corrected spectrum without installing software, or when
  NUS 2D/pseudo-3D/3D data requires direct-FT or iterative SMILE reconstruction.
when_to_use_negative:
- Input is already a processed frequency-domain spectrum that only needs peak picking
  — use DEEP Picker/Voigt Fitter directly rather than the FID-processing pipeline
- Data is static 1D ^1H spectra not requiring NUS reconstruction or pseudo-3D series
  handling — simpler desktop tools may suffice
- Local offline use is required but the browser blocks WebWorker/WebAssembly from
  file:// origins and browser security settings cannot be modified
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3420
tools:
- name: COLMARvista
  role: Primary client-side web application for NMR FID processing, visualization,
    and spectral analysis
  repo: https://github.com/lidawei1975/colmarvista
- name: SMILE WebAssembly plugin
  role: Proprietary plugin for fast direct-FT and iterative SMILE reconstruction of
    NUS 2D, pseudo-3D, and 3D NMR data
  repo: https://github.com/lidawei1975/colmarvista
- name: DEEP Picker
  role: AI-driven deep neural network for automated peak deconvolution of complex
    2D spectra after processing
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Voigt/Gaussian 2D line-shape fitting of picked peaks for quantitative analysis
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:8d113ff8817c5aac2e3a032214fc9bc5fc8fb97e8fd640c1d25378743d1cd624
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
  - WebAssembly
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6206896551724138
n_claims: 29
---

# nmr-spectral-processing

## Summary

Browser-based processing of 1D/2D/pseudo-3D NMR spectra using COLMARvista, which processes raw Bruker time-domain FID data (fid, ser) entirely client-side via WebAssembly/JavaScript with automated phase correction, apodization, zero filling, solvent suppression, and baseline correction.

## When to use

Use when raw Bruker fid/ser time-domain NMR FID data needs processing into a phased, baseline-corrected spectrum without installing software, or when NUS 2D/pseudo-3D/3D data requires direct-FT or iterative SMILE reconstruction.

## When NOT to use

- Input is already a processed frequency-domain spectrum that only needs peak picking — use DEEP Picker/Voigt Fitter directly rather than the FID-processing pipeline
- Data is static 1D ^1H spectra not requiring NUS reconstruction or pseudo-3D series handling — simpler desktop tools may suffice
- Local offline use is required but the browser blocks WebWorker/WebAssembly from file:// origins and browser security settings cannot be modified

## Inputs

- Raw Bruker time-domain FID data (fid, ser files)
- NUS sampling schedule for non-uniformly sampled 2D/pseudo-3D/3D experiments
- Reference peak lists (.list, .tab, .txt, .csv) for assignment transfer

## Outputs

- Processed, uniformly sampled 2D/pseudo-3D spectrum (phased, apodized, zero-filled, baseline-corrected)
- SMILE-reconstructed NUS spectrum (uniformly sampled time domain or frequency-domain spectrum)
- Picked/deconvoluted peak lists with Voigt/Gaussian line-shape parameters

## How to apply

Load the raw Bruker fid or ser FID files into COLMARvista (visit https://lidawei1975.github.io/colmarvista/ or run locally by opening index.html); all processing happens client-side so data never leaves the computer. For uniformly sampled data, apply the automated pipeline: phase correction, apodization, zero filling, solvent (water) suppression, baseline correction, and Fourier transformation. For non-uniformly sampled (NUS) 2D, pseudo-3D, or 3D experiments, first apply fast direct-FT reconstruction for a first-pass spectrum, then run the proprietary SMILE WebAssembly plugin (iterative sparse lineshape-enhanced reconstruction, distributed under a GPL V3 linking exception with permission from its original author) for higher fidelity. Optionally integrate DEEP Picker (deep neural network) or Simple Picker (local maxima) with Voigt/Gaussian deconvolution via Voigt Fitter for automated peak deconvolution. Export the reconstructed spectrum for downstream viewing and analysis.

## Related tools

- **COLMARvista** (Primary client-side web application for NMR FID processing, visualization, and spectral analysis) — https://github.com/lidawei1975/colmarvista
- **SMILE WebAssembly plugin** (Proprietary plugin for fast direct-FT and iterative SMILE reconstruction of NUS 2D, pseudo-3D, and 3D NMR data) — https://github.com/lidawei1975/colmarvista
- **DEEP Picker** (AI-driven deep neural network for automated peak deconvolution of complex 2D spectra after processing) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Voigt/Gaussian 2D line-shape fitting of picked peaks for quantitative analysis) — https://github.com/lidawei1975/colmarvista

## Examples

```
Visit https://lidawei1975.github.io/colmarvista/ in your browser, load the Bruker fid/ser FID plus NUS schedule, then run fast direct-FT followed by iterative SMILE reconstruction; for offline use open index.html (launch Chrome with --allow-file-access-from-files to enable WebAssembly)
```

## Evaluation signals

- Spectrum is correctly phased and baseline-corrected after the automated pipeline (compare before/after phase correction visually in COLMARvista's WebGL contour display)
- NUS-reconstructed spectrum shows no reconstructed-peak artifacts; SMILE iterative output should be higher fidelity than the fast direct-FT first pass
- DEEP Picker/Voigt Fitter deconvolution resolves overlapping cross-peaks consistent with published quantitative behavior (compound concentration determination, cross-peak specific relaxation parameters)
- Zero-filling, apodization, and water suppression steps applied and reflected in 1D orthogonal projections/slices
- Exported spectrum matches expected dimensionality (2D, pseudo-3D, or 3D) and all data remained local (server accepts no uploads)

## Limitations

- The SMILE WebAssembly plugin is proprietary and distributed only under a GPL V3 linking exception, used with permission from its original author
- Local (offline) use requires opening index.html and manually weakening browser security (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari 'Disable Local File Restrictions'), which poses a security risk
- WebWorker and WebAssembly cannot be loaded automatically when run locally unless browser settings are modified
- COLMARvista is a static web server: no server-side processing; the browser must support WebGL for high-performance contour rendering

## Evidence
