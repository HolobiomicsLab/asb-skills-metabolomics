---
name: fourier-transform-signal-reconstruction
description: Use when raw time-domain NMR FID data (Bruker `fid`/`ser`) comes with
  a non-uniform sampling schedule and a uniformly sampled 2D, pseudo-3D, or 3D spectrum
  must be reconstructed before further processing or peak analysis.
when_to_use_negative:
- The input is already uniformly sampled and fully processed frequency-domain data
  — direct Fourier transformation of conventionally acquired data suffices without
  SMILE iteration
- The goal is peak deconvolution or line-shape fitting on an existing spectrum (use
  DEEP Picker/Voigt Fitter instead of reconstruction)
- Data are 1D spectra without a NUS schedule, where reconstruction offers no benefit
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3419
tools:
- name: COLMARvista
  role: Web-based client-side pipeline that loads the NUS FID data, runs direct-FT/SMILE
    reconstruction, and passes the result to phase correction, apodization, zero filling,
    solvent suppression, and baseline correction
  repo: https://github.com/lidawei1975/colmarvista
- name: SMILE WebAssembly plugin
  role: Proprietary plugin performing the iterative sparse multidimensional iterative
    lineshape-enhanced (SMILE) NUS reconstruction; distributed under a GPL V3 linking
    exception with the original author's permission
  repo: https://github.com/lidawei1975/colmarvista
- name: WebAssembly
  role: Client-side runtime enabling direct in-browser processing of raw Bruker time-domain
    FID data without uploads
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:6a6d6dce44bb1dbf3c334d8ba8c72f8ba604bfcbd20329f960ed89fa8289f55e
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - SMILE WebAssembly plugin
  - WebAssembly
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3419
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# fourier-transform-signal-reconstruction

## Summary

Reconstructs non-uniformly sampled (NUS) multidimensional NMR spectra using fast direct-FT for a first-pass uniformly sampled spectrum and iterative SMILE reconstruction for higher fidelity, as implemented in COLMARvista's WebAssembly pipeline for Bruker `fid`/`ser` FID data.

## When to use

Use when raw time-domain NMR FID data (Bruker `fid`/`ser`) comes with a non-uniform sampling schedule and a uniformly sampled 2D, pseudo-3D, or 3D spectrum must be reconstructed before further processing or peak analysis.

## When NOT to use

- The input is already uniformly sampled and fully processed frequency-domain data — direct Fourier transformation of conventionally acquired data suffices without SMILE iteration
- The goal is peak deconvolution or line-shape fitting on an existing spectrum (use DEEP Picker/Voigt Fitter instead of reconstruction)
- Data are 1D spectra without a NUS schedule, where reconstruction offers no benefit

## Inputs

- Raw Bruker time-domain 2D FID data (`fid`, `ser` files) from NUS experiments
- NUS sampling schedule accompanying the FID data
- Pseudo-3D or 3D NUS experiment data

## Outputs

- Reconstructed uniformly sampled 2D spectrum (exportable file for downstream viewing and analysis)
- First-pass direct-FT spectrum for quick inspection

## How to apply

Load the raw NUS time-domain FID data (Bruker `fid`/`ser` format) together with the NUS sampling schedule into COLMARvista, which processes everything client-side via WebAssembly. First apply the fast direct-FT reconstruction to obtain a first-pass uniformly sampled spectrum; if higher fidelity is needed, run the iterative SMILE reconstruction (the sparse multidimensional iterative lineshape-enhanced method of Ying et al., J Biomol NMR 2017). The reconstruction feeds into the downstream client-side pipeline of automated phase correction, apodization, zero filling, solvent suppression, and baseline correction, and the reconstructed uniformly sampled 2D spectrum can then be exported for viewing or analysis. SMILE is distributed as a proprietary WebAssembly plugin under a linking exception to COLMARvista's GPL V3 license, used with permission from its original author.

## Related tools

- **COLMARvista** (Web-based client-side pipeline that loads the NUS FID data, runs direct-FT/SMILE reconstruction, and passes the result to phase correction, apodization, zero filling, solvent suppression, and baseline correction) — https://github.com/lidawei1975/colmarvista
- **SMILE WebAssembly plugin** (Proprietary plugin performing the iterative sparse multidimensional iterative lineshape-enhanced (SMILE) NUS reconstruction; distributed under a GPL V3 linking exception with the original author's permission) — https://github.com/lidawei1975/colmarvista
- **WebAssembly** (Client-side runtime enabling direct in-browser processing of raw Bruker time-domain FID data without uploads)

## Evaluation signals

- The direct-FT first-pass spectrum shows all expected cross-peaks of the 2D/pseudo-3D experiment at the scheduled NUS indirect-dimension positions
- Iterative SMILE output shows reduced artifacts and cleaner baseline relative to the direct-FT first pass, justifying the higher-fidelity run
- The reconstructed spectrum is uniformly sampled on a regular grid and successfully enters the automated phase correction, apodization, zero filling, solvent suppression, and baseline correction steps without format errors
- All processing stayed local: the browser-based static-server workflow performed reconstruction without uploading the FID data

## Limitations

- The SMILE plugin is proprietary — it is distributed under a linking exception to the GPL V3 open-source license, not as fully open-source code
- Local (offline) use requires modifying browser security settings (e.g. Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false) to load WebWorker/WebAssembly, which poses a security risk
- Reconstruction is described for 2D, pseudo-3D, and 3D NUS experiments; higher-dimensional or non-Bruker raw formats are not covered by the documented workflow

## Evidence
