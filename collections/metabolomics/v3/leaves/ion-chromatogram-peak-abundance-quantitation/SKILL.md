---
name: ion-chromatogram-peak-abundance-quantitation
description: Use when MRM ion chromatograms contain coeluted peaks (2–3 isomers sharing
  one chromatogram) and per-compound peak abundances must be computed automatically
  rather than attributed wholly to one targeted compound.
when_to_use_negative:
- The chromatogram contains a single well-separated peak with no coelution — simpler
  integration suffices
- The input is already a processed feature/abundance table rather than raw or converted
  MRM chromatogram data
- Saturated peaks needing regression-based absolute quantitation are the target —
  those are excluded from regression rather than deconvolved
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Automated tool performing peak detection, Gaussian deconvolution of coeluted
    peaks, and abundance quantitation of MRM data
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment in which MRMQuant's deconvolution and findpeaks-based
    peak detection execute
- name: findpeaks
  role: MATLAB built-in function supplying the possible peak tip locations used to
    initialize Gaussian models
- name: Microsoft Excel
  role: Output format for the per-peak quantitation results (xlsx)
- name: ProteoWizard
  role: msconvert converts vendor raw MRM data into mzML/mzXML for input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:784a2dfc6c3dc2fdec714fb51b29b29df6f2e0ac5646031d6c8158e88adef242
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# ion-chromatogram-peak-abundance-quantitation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-peak abundances from MRM ion chromatograms by fitting Gaussian models at findpeaks-detected tip locations and deconvolving coeluted peaks so each isomer receives its own abundance value.

## When to use

Use when MRM ion chromatograms contain coeluted peaks (2–3 isomers sharing one chromatogram) and per-compound peak abundances must be computed automatically rather than attributed wholly to one targeted compound.

## When NOT to use

- The chromatogram contains a single well-separated peak with no coelution — simpler integration suffices
- The input is already a processed feature/abundance table rather than raw or converted MRM chromatogram data
- Saturated peaks needing regression-based absolute quantitation are the target — those are excluded from regression rather than deconvolved

## Inputs

- Smoothed MRM ion chromatogram signal per transition
- Candidate peak tip locations from MATLAB findpeaks
- Method file with targeted compound information (precursor/product m/z, RT)
- MRM sample files (mzML/mzXML, converted from vendor raw data via ProteoWizard msconvert)

## Outputs

- Per-peak fitted height and sigma (Gaussian parameters) for each deconvolved peak
- Compound abundance values integrated from the fitted Gaussian signals after background removal
- Quantitation results table in Microsoft Excel xlsx format

## How to apply

Load the smoothed ion chromatogram signal and candidate peak tip locations identified by MATLAB's built-in findpeaks from the MRM sample data. Construct the coeluted signal union across overlapping transitions for the targeted compound region. Initialize Gaussian models at the findpeaks tip locations and iteratively adjust each peak's height and sigma (width) so the summed Gaussians minimize the sum of differences against the extracted union signal. Extract each Gaussian's fitted height (after background removal) as that compound's abundance and write results to the quantitation output table (xlsx). This differs from baseline-connecting approaches like TargetLynx, which attribute all coeluted peaks to the targeted compound and overestimate its abundance.

## Related tools

- **MRMQuant** (Automated tool performing peak detection, Gaussian deconvolution of coeluted peaks, and abundance quantitation of MRM data) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment in which MRMQuant's deconvolution and findpeaks-based peak detection execute)
- **findpeaks** (MATLAB built-in function supplying the possible peak tip locations used to initialize Gaussian models)
- **Microsoft Excel** (Output format for the per-peak quantitation results (xlsx))
- **ProteoWizard** (msconvert converts vendor raw MRM data into mzML/mzXML for input)

## Evaluation signals

- Each coeluted isomer receives a distinct abundance rather than one compound absorbing all coeluted signal (contrast: TargetLynx attributed all peaks to PC 34:3, overestimating its abundance)
- Fitted sum of Gaussians minimizes the sum of differences with the extracted signal union after background removal
- Low average STD and RT differences (<0.1) across technical replicate QC injections for the same peak
- Consistent abundance for the same peak when multiple peaks exist near the specified RT (OpenMS failed this check with large abundance variations)
- Results table exported as xlsx contains fitted height and sigma per deconvolved peak

## Limitations

- Deconvolution assumes Gaussian peak shapes; peaks with non-Gaussian or unevenly sampled signals are problematic (MRMkit and Skyline showed problematic quantitation of coeluted peaks, noisy peaks, and unevenly sampled signals)
- When multiple peaks exist in the neighborhood of the specified RT, peak identity consistency can fail (OpenMS produced large abundance variations in this case)
- Baseline estimation approaches (e.g., TargetLynx connecting nearby valleys) can underestimate abundance when coelution occurs in the right valley

## Evidence

- [results] Gaussian deconvolution finding: [evidence span withheld — anchor status: not_found]
- [abstract] findpeaks peak detection: [evidence span withheld — anchor status: not_found]
