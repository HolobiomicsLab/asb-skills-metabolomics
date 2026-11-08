---
name: coeluted-peak-fitting
description: Use when an MRM ion chromatogram contains overlapping/coeluted peaks
  for a targeted compound (e.g., isomers sharing a transition, like the PC 34:3 isomers
  at RTs 4.72, 4.84, and 4.92 min) and per-peak abundances must be separated.
when_to_use_negative:
- Peaks are baseline-resolved with no coelution — simple integration suffices and
  Gaussian fitting adds assumptions
- The chromatogram is dominated by noise or has unevenly sampled signal, where deconvolution
  fits are unreliable (MRMkit and Skyline showed problematic quantitation of noisy
  peaks and peaks with unevenly sampled signals)
- The goal is batch-effect correction or standard-curve regression rather than peak
  separation — other MRMQuant workflow steps apply
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Implements the automatic Gaussian deconvolution of coeluted peaks as part
    of its MRM quantitation pipeline
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function used to find possible peak tip locations that seed
    the Gaussian models
- name: MATLAB
  role: Runtime environment in which the smoothing, findpeaks detection, and Gaussian
    fitting are executed
- name: Microsoft Excel
  role: Format of the quantitation output table receiving per-peak abundance results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:9949959b95f0e01ea4a56d52d2df702343b782addb106c4d18d393d07165424d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MATLAB
  - MRMQuant
  - Microsoft Excel
  - findpeaks
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# coeluted-peak-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Gaussian-based deconvolution of coeluted peaks in MRM ion chromatograms, as implemented in MRMQuant: fit overlapping Gaussian models at detected peak tips so the summed signals minimize the difference to the extracted signal union, then use each fitted peak height as the compound's abundance.

## When to use

Use when an MRM ion chromatogram contains overlapping/coeluted peaks for a targeted compound (e.g., isomers sharing a transition, like the PC 34:3 isomers at RTs 4.72, 4.84, and 4.92 min) and per-peak abundances must be separated.

## When NOT to use

- Peaks are baseline-resolved with no coelution — simple integration suffices and Gaussian fitting adds assumptions
- The chromatogram is dominated by noise or has unevenly sampled signal, where deconvolution fits are unreliable (MRMkit and Skyline showed problematic quantitation of noisy peaks and peaks with unevenly sampled signals)
- The goal is batch-effect correction or standard-curve regression rather than peak separation — other MRMQuant workflow steps apply

## Inputs

- Smoothed MRM ion chromatogram signals (sample files, e.g., mzML converted via ProteoWizard msconvert)
- Candidate peak tip locations from MATLAB findpeaks
- Method file with targeted compound information (precursor/product m/z, RT)
- Coeluted signal union across overlapping transitions for the targeted compound region

## Outputs

- Fitted Gaussian parameters (height and sigma) per peak
- Per-peak abundance values written to the quantitation output table (Microsoft Excel xlsx)

## How to apply

Load the smoothed ion chromatogram signal and candidate peak tip locations identified by MATLAB's built-in findpeaks from the MRM sample data. Construct the coeluted signal union across overlapping transitions for the targeted compound region. Initialize Gaussian models at the findpeaks tip locations and iteratively adjust each peak's height and sigma (width) so that the sum of differences between the summed Gaussians and the extracted union signal (after background removal) is minimized. Extract each Gaussian's fitted height as its abundance and write per-peak abundances to the quantitation output table (xlsx). This deconvolution is why MRMQuant can attribute coeluted peaks to individual isomers, whereas tools like TargetLynx attributed all coeluted PC 34:3 peaks to one compound and overestimated abundance.

## Related tools

- **MRMQuant** (Implements the automatic Gaussian deconvolution of coeluted peaks as part of its MRM quantitation pipeline) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function used to find possible peak tip locations that seed the Gaussian models)
- **MATLAB** (Runtime environment in which the smoothing, findpeaks detection, and Gaussian fitting are executed)
- **Microsoft Excel** (Format of the quantitation output table receiving per-peak abundance results)

## Evaluation signals

- Each fitted Gaussian's summed signal minimizes the sum of differences against the extracted signal union after background removal
- Number of fitted Gaussians matches the number of findpeaks tip locations in the coeluted region (e.g., three for the PC 34:3 isomers at 4.72, 4.84, and 4.92 min)
- Coeluted peaks are attributed to distinct isomers rather than collapsed onto one compound (unlike TargetLynx, which attributed all coeluted PC 34:3 peaks to one compound and overestimated abundance)
- Fitted peak heights are consistent across technical replicates (low average STD and RT differences across replicates)
- Per-peak abundances appear in the xlsx quantitation output table for each targeted compound

## Limitations

- Assumes Gaussian peak shapes; non-Gaussian or heavily distorted coeluted signals may not fit well
- Baseline estimation issues under coelution can bias abundance (TargetLynx underestimated abundance due to high coelution in the right valley, showing baseline sensitivity in this regime)
- Tools with low replicate STD (MRMkit, Skyline) still had problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals, so fit quality should be visually inspected and manually corrected if required

## Evidence

- [results] For coeluted peaks, MRMQuant assumes Gaussian shapes and solves for each peak's height and sigma so that the combined signals minimize the sum of differences with the extracted signal union: [evidence span withheld — anchor status: not_found]
- [abstract] In MRMQuant, the MATLAB built-in function "findpeaks" is used to find possible peak tip locations: [evidence span withheld — anchor status: not_found]
