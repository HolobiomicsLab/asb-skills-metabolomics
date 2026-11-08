---
name: nmr-cross-peak-overlap-resolution
description: Use when a processed 2D or pseudo-3D NMR spectrum contains significantly
  overlapped cross-peaks that prevent straightforward peak picking, and quantitatively
  analyzable individual components (positions, line shapes, intensities) are needed.
when_to_use_negative:
- The spectrum is 1D or the goal is raw FID processing (phase correction, apodization,
  Fourier transformation) rather than 2D cross-peak deconvolution
- The spectrum is already fully deconvoluted with a clean, non-overlapping peak list
  — Simple Picker local-maxima picking suffices
- Data must not leave the local machine and the browser cannot run WebWorker/WebAssembly
  locally without the documented browser flag changes
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: DEEP Picker
  role: Deep-neural-network peak picker that detects candidate cross-peak positions
    and drives automated resolution of overlapping cross-peaks into individual components
  repo: https://github.com/lidawei1975/colmarvista
- name: Voigt Fitter
  role: Performs Voigt/Gaussian line-shape deconvolution and 2D fitting of the resolved
    components, reconstructing the experimental spectrum for quantitative analysis
  repo: https://github.com/lidawei1975/colmarvista
- name: COLMARvista
  role: Web-based host environment that integrates DEEP Picker and Voigt Fitter, runs
    the deconvolution client-side in the browser, and writes the final deconvoluted
    peak list
  repo: https://github.com/lidawei1975/colmarvista
- name: Simple Picker
  role: Local-maxima peak picker whose output is compared/combined with the DEEP Picker
    list to handle residual ambiguities after deconvolution
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:fc573751e151ab77192e473cb8c21d08b4fb9a5c0d4d4f33753d8d1e4d8ec08d
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nmr-cross-peak-overlap-resolution

## Summary

Resolves overlapping 2D NMR cross-peaks into individually quantifiable components by feeding processed 2D spectra through COLMARvista's integrated DEEP Picker deep-neural-network peak picker and Voigt Fitter line-shape deconvolution, enabling quantitative analysis such as compound concentration determination and cross-peak-specific relaxation parameter extraction.

## When to use

Use when a processed 2D or pseudo-3D NMR spectrum contains significantly overlapped cross-peaks that prevent straightforward peak picking, and quantitatively analyzable individual components (positions, line shapes, intensities) are needed.

## When NOT to use

- The spectrum is 1D or the goal is raw FID processing (phase correction, apodization, Fourier transformation) rather than 2D cross-peak deconvolution
- The spectrum is already fully deconvoluted with a clean, non-overlapping peak list — Simple Picker local-maxima picking suffices
- Data must not leave the local machine and the browser cannot run WebWorker/WebAssembly locally without the documented browser flag changes

## Inputs

- Processed 2D (or pseudo-3D series of 2D planes) NMR spectrum with overlapping cross-peaks
- Optionally, a local-maxima (Simple Picker) peak list for cross-checking DEEP Picker results
- Reference assignment lists (.list, .tab, .txt, .csv) for downstream assignment transfer after deconvolution

## Outputs

- Deconvoluted per-spectrum peak list of individually resolved cross-peak components
- Reconstructed experimental spectrum built from fitted Voigt/Gaussian line-shape components
- Quantitative per-peak parameters (e.g., intensities for concentration determination, cross-peak-specific relaxation parameters)

## How to apply

Load a processed 2D spectrum file into COLMARvista and feed it to the DEEP Picker deep-neural-network peak picker, which generates an initial peak list of candidate cross-peak positions. Run the automated neural-network-based deconvolution to resolve overlapping cross-peaks into individual components, then reconstruct the experimental spectrum from the fitted components. Optionally refine the resolved components by comparing or combining the DEEP Picker output with the local-maxima Simple Picker output to handle residual ambiguities. Finally write the resolved per-spectrum peak list as the deconvoluted output, which can feed downstream fitting (e.g., Voigt/Gaussian line-shape fitting across relaxation T1/T2, titration, or kinetics series) for concentration or relaxation-parameter extraction.

## Related tools

- **DEEP Picker** (Deep-neural-network peak picker that detects candidate cross-peak positions and drives automated resolution of overlapping cross-peaks into individual components) — https://github.com/lidawei1975/colmarvista
- **Voigt Fitter** (Performs Voigt/Gaussian line-shape deconvolution and 2D fitting of the resolved components, reconstructing the experimental spectrum for quantitative analysis) — https://github.com/lidawei1975/colmarvista
- **COLMARvista** (Web-based host environment that integrates DEEP Picker and Voigt Fitter, runs the deconvolution client-side in the browser, and writes the final deconvoluted peak list) — https://github.com/lidawei1975/colmarvista
- **Simple Picker** (Local-maxima peak picker whose output is compared/combined with the DEEP Picker list to handle residual ambiguities after deconvolution) — https://github.com/lidawei1975/colmarvista

## Examples

```
open https://lidawei1975.github.io/colmarvista/ in browser; load processed 2D spectrum; run DEEP Picker then Voigt/Gaussian deconvolution
```

## Evaluation signals

- The reconstructed spectrum assembled from the resolved components matches the experimental 2D spectrum (deconvolution/reconstruction consistency)
- Every significantly overlapped cross-peak region yields a distinct resolved component rather than being merged into one peak
- Per-peak Voigt/Gaussian fit parameters produce physically meaningful quantities (e.g., compound concentrations, T1/T2 relaxation parameters across pseudo-3D series planes)
- DEEP Picker peak list and Simple Picker local-maxima list agree on well-isolated peaks; discrepancies only occur in overlapped regions flagged for refinement
- Output is a per-spectrum peak list file in a format loadable for downstream assignment transfer (compatible with .list/.tab/.txt/.csv reference lists)

## Limitations

- Severely overlapped regions may leave residual ambiguities that require combining the DEEP Picker output with the local-maxima Simple Picker output
- Local (offline) use requires modifying browser security settings (e.g., Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false), which the README warns poses a security risk
- Deconvolution quality depends on prior processing steps (phase correction, apodization, zero filling, solvent suppression, baseline correction) being performed correctly

## Evidence
