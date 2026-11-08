---
name: gaussian-peak-deconvolution
description: Use when an ion chromatogram's targeted compound peak is coeluted with
  other peaks (e.g., isomers sharing a transition with close RTs) and per-peak abundances
  must be separated rather than summed.
when_to_use_negative:
- The targeted compound peak is isolated with no coelution — simple integration/baseline
  methods suffice
- Peaks are so noisy or unevenly sampled that Gaussian shape assumptions fail (MRMkit
  and Skyline had problematic quantitation of noisy peaks and peaks with unevenly
  sampled signals)
- A saturated peak — saturated signals are excluded from regression in MRMQuant as
  unreliable, so deconvolved abundances would not be trustworthy
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Hosts the automatic deconvolution of coeluted peaks and writes per-peak abundances
    to the xlsx quantitation table
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function that supplies the initial peak tip locations at which
    the Gaussian models are initialized
- name: MATLAB
  role: Runtime environment in which the Gaussian fitting/minimization is executed
- name: Microsoft Excel
  role: Output format (xlsx) for the deconvolved per-peak abundance results
- name: ProteoWizard
  role: msconvert converts vendor raw MRM data to the formats MRMQuant loads before
    deconvolution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:30f5c5b1f7c51d4d489dcb50ee3692063861d867c911062dcf720dfc4c2f6add
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
  - ProteoWizard
  - findpeaks
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gaussian-peak-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic deconvolution of coeluted peaks in MRM ion chromatograms: MRMQuant fits Gaussian models (height + sigma) at findpeaks tip locations so the summed Gaussians minimize the sum of differences with the extracted coeluted signal union, then integrates each fitted Gaussian (after background removal) as the compound's abundance.

## When to use

Use when an ion chromatogram's targeted compound peak is coeluted with other peaks (e.g., isomers sharing a transition with close RTs) and per-peak abundances must be separated rather than summed.

## When NOT to use

- The targeted compound peak is isolated with no coelution — simple integration/baseline methods suffice
- Peaks are so noisy or unevenly sampled that Gaussian shape assumptions fail (MRMkit and Skyline had problematic quantitation of noisy peaks and peaks with unevenly sampled signals)
- A saturated peak — saturated signals are excluded from regression in MRMQuant as unreliable, so deconvolved abundances would not be trustworthy

## Inputs

- Smoothed MRM ion chromatogram signal per transition (mzML or converted vendor raw data)
- Candidate peak tip locations from MATLAB findpeaks
- Method file with targeted compound information (precursor/product m/z, expected RT)

## Outputs

- Per-Gaussian fitted parameters (height and sigma) for each deconvolved peak
- Per-peak abundance values integrated from the background-removed fitted Gaussians
- Rows in the MRMQuant quantitation output table (xlsx)

## How to apply

Load the smoothed ion chromatogram signal and the candidate peak tip locations identified by the MATLAB built-in function findpeaks from the MRM sample data. Construct the coeluted signal union across overlapping transitions for the targeted compound region. Initialize Gaussian models at the findpeaks tip locations and iteratively adjust each peak's height and sigma (standard deviation/width) so the sum of differences between the summed Gaussians and the extracted union signal is minimized. After background removal, integrate each fitted Gaussian signal as that peak's abundance and write per-peak abundances to the quantitation output table (xlsx via Microsoft Excel). This resolves cases like three PC 34:3 isomers with RTs at 4.72, 4.84, and 4.92 min, where naive baseline-based tools over- or underestimate abundance.

## Related tools

- **MRMQuant** (Hosts the automatic deconvolution of coeluted peaks and writes per-peak abundances to the xlsx quantitation table) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function that supplies the initial peak tip locations at which the Gaussian models are initialized)
- **MATLAB** (Runtime environment in which the Gaussian fitting/minimization is executed)
- **Microsoft Excel** (Output format (xlsx) for the deconvolved per-peak abundance results)
- **ProteoWizard** (msconvert converts vendor raw MRM data to the formats MRMQuant loads before deconvolution)

## Evaluation signals

- The summed fitted Gaussians minimize the sum of differences with the extracted coeluted signal union — residual should approach the background level
- Each detected tip (e.g., three isomer RTs at 4.72, 4.84, and 4.92 min) is attributed to its own Gaussian, not collapsed onto one compound (contrast: TargetLynx attributed all coeluted peaks to PC 34:3, overestimating its abundance)
- Abundance for each peak equals the integral of its fitted Gaussian after background removal, not the valley-to-valley area (TargetLynx's valley-connecting baseline underestimated abundance under right-valley coelution)
- Per-peak abundances appear as individual rows in the xlsx quantitation table
- Across technical replicates, deconvolved abundances and RTs should show low variability (comparable tools showed low (<0.1) average STD and RT differences when deconvolution worked)

## Limitations

- Assumes Gaussian peak shapes; highly asymmetric or unevenly sampled coeluted signals may not fit
- Fitting quality depends on findpeaks correctly locating all tip locations; missed tips yield missing Gaussians
- Only MRMQuant performs this deconvolution among the compared tools — MRMkit, Skyline, TargetLynx, and OpenMS had problematic or inconsistent quantitation of coeluted peaks

## Evidence

- [results] Coeluted peak Gaussian fitting workflow: [evidence span withheld — anchor status: not_found]
- [abstract] findpeaks initialization: [evidence span withheld — anchor status: not_found]
