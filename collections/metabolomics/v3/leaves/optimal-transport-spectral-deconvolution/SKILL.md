---
name: optimal-transport-spectral-deconvolution
description: Use when 1H NMR mixture spectra have strongly overlapping peaks, distorted
  lineshapes (bad shimming), temperature- or solvent-induced peak shifts, or low-resolution
  benchtop acquisition where peak-picking quantification fails.
when_to_use_negative:
- The mixture contains unknown components absent from the spectral library, since
  optimal transport deconvolution assigns signal only to supplied library spectra.
- Absolute quantification is required without a quantitative external-standard spectrum
  algebraically added to the mixture spectrum.
- Peaks are well resolved and peak-picking tools suffice; note MANIQ outperformed
  Magnetstein in Experiment 2 (AvRE 1.0% vs 3.0%).
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: Magnetstein
  role: Primary tool implementing optimal-transport-based quantification of 1H NMR
    mixtures with kappa_mixture/kappa_components tuning
  repo: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: Peak-based competitor used as an AvRE benchmark in the comparison table
- name: ACD/Spectrus-Target-Protected Profiling
  role: Commercial peak-based competitor (ACD/TP) used as an AvRE benchmark
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:036a6c07c28d6147ca46d58a00b67ab947c0702163968e0c70c72593b054d5bb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  repo_url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - ACD/Spectrus-Target-Protected Profiling
  - MANIQ
  - Magnetstein
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# optimal-transport-spectral-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify mixture components in 1H NMR spectra by treating mixture and library spectra as probability distributions of signal intensity and solving an optimal transport (Wasserstein) deconvolution with Magnetstein, using tunable sharpness parameters kappa_mixture and kappa_components.

## When to use

Use when 1H NMR mixture spectra have strongly overlapping peaks, distorted lineshapes (bad shimming), temperature- or solvent-induced peak shifts, or low-resolution benchtop acquisition where peak-picking quantification fails.

## When NOT to use

- The mixture contains unknown components absent from the spectral library, since optimal transport deconvolution assigns signal only to supplied library spectra.
- Absolute quantification is required without a quantitative external-standard spectrum algebraically added to the mixture spectrum.
- Peaks are well resolved and peak-picking tools suffice; note MANIQ outperformed Magnetstein in Experiment 2 (AvRE 1.0% vs 3.0%).

## Inputs

- 1H NMR mixture spectrum (e.g., high-resolution or 43 MHz Magritek benchtop)
- Reference library spectra of each known mixture component (e.g., alpha-pinene, limonene, leucine, isoleucine, valine)
- Known gravimetric composition of the mixture (for error computation)
- Tunable sharpness parameters kappa_mixture and kappa_components

## Outputs

- Estimated relative/molar concentrations pi per component
- Per-component relative errors (RE) and average relative error (AvRE) against gravimetric composition
- Comparison table of AvRE versus MANIQ and ACD/TP (e.g., avre_comparison.csv)

## How to apply

Load the mixture spectrum together with reference library spectra of the known components and treat both as probability distributions of signal intensity, as prescribed by Magnetstein's formalism. Run Magnetstein's optimal-transport deconvolution against the component library with default kappa_mixture and kappa_components unless a tuning step is justified. Decision points: for low-field benchtop data (e.g., the 43 MHz Magritek Experiment 3 fragrance mixture of alpha-pinene and limonene), set kappa_components higher than kappa_mixture, which the paper reports as optimal; for spectra contaminated by extra resonances, lower kappa_mixture from the default so contaminating peaks can be excluded; avoid very small kappa_mixture, which pushes the smaller component into the vortex omega and inflates error. Extract the estimated relative concentrations pi and compute per-component relative errors (RE) and the average relative error (AvRE) against gravimetrically known compositions. Benchmark against peak-based competitors MANIQ and ACD/Spectrus-Target-Protected Profiling (ACD/TP), which run with default settings.

## Related tools

- **Magnetstein** (Primary tool implementing optimal-transport-based quantification of 1H NMR mixtures with kappa_mixture/kappa_components tuning) — https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (Peak-based competitor used as an AvRE benchmark in the comparison table)
- **ACD/Spectrus-Target-Protected Profiling** (Commercial peak-based competitor (ACD/TP) used as an AvRE benchmark)

## Evaluation signals

- Reproduced AvRE matches article values: Experiment 1 = 0.0%, Experiment 2 = 3.0%, Experiment 3 = 9.0%, Experiment 4 = 3.6%, Experiment 5 = 3.6%.
- Experiment 3 per-component REs equal 0.0% for alpha-pinene and 33.3% for limonene when kappa_components > kappa_mixture.
- Reproduced AvRE outperforms ACD/TP and MANIQ in Experiments 1, 3, 4, and 5 (MANIQ leads only in Experiment 2).
- Output CSV contains proportions with relative errors within reported tolerances (e.g., BCAA supplement: Ile 0.25 +/- 4.2%, Leu 0.25 +/- 4.2%, Val 0.25 +/- 10.7%).
- Sensitivity check: very small kappa_mixture should reproduce the documented failure mode where the smaller component is assigned to vortex omega with large error.

## Limitations

- Small kappa_mixture values cause the method to put the smaller component into vortex omega, causing large relative error.
- Default kappa_mixture can degrade performance when contaminating/degradation peaks are present; lowering it is needed (Experiment 2).
- Benchtop low-resolution regime gives higher errors (Experiment 3 AvRE 9.0%, limonene RE 33.3%) than high-resolution data.
- Best kappa settings are experiment-dependent: kappa_components > kappa_mixture was optimal only for the benchtop Experiment 3.

## Evidence
