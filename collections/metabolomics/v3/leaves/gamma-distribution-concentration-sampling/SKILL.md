---
name: gamma-distribution-concentration-sampling
description: Use when you must synthesize annotated training NMR spectra (e.g., to
  train a CRNN model like NMRQNet) and need realistic metabolite mixture compositions
  without measured ground-truth concentrations.
when_to_use_negative:
- You have real, measured plasma spectra with expert-annotated concentrations (e.g.,
  Chenomx fitting reports) and do not need synthetic training data
- You need actual biological concentration estimates — gamma-drawn values are synthetic
  training labels, not measured quantifications
- You are fitting a spectrum as a linear combination of library references (e.g.,
  ASICS-style regression) rather than generating training mixtures
tools:
- name: NMRQNet
  role: Provides the metabolite reference spectral library, the Lorentzian-curve spectrum
    rendering, and the simulated-data pipeline whose concentration sampling step this
    skill describes; output csv must match NMRQNet's input convention.
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:b3423140864b044a9775e5b38307d58219e9d2f5c85899adb3a4b96d7601d297
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gamma-distribution-concentration-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A synthetic-data generation step in the NMRQNet training pipeline that samples metabolite relative concentrations from a gamma distribution to build realistic, non-negative 1H plasma mixture spectra when no annotated NMR spectra database exists.

## When to use

Use when you must synthesize annotated training NMR spectra (e.g., to train a CRNN model like NMRQNet) and need realistic metabolite mixture compositions without measured ground-truth concentrations.

## When NOT to use

- You have real, measured plasma spectra with expert-annotated concentrations (e.g., Chenomx fitting reports) and do not need synthetic training data
- You need actual biological concentration estimates — gamma-drawn values are synthetic training labels, not measured quantifications
- You are fitting a spectrum as a linear combination of library references (e.g., ASICS-style regression) rather than generating training mixtures

## Inputs

- NMRQNet metabolite reference spectral library (peak signatures: cluster, peak position, width, height)
- Tutorial example spectrum NMRQNet/data/tutorial_data/control_human_plasma.csv to fix the chemical shift axis convention (first column = shift values, each subsequent column = one sample)
- Gamma distribution parameters for metabolite relative concentration sampling

## Outputs

- synthetic_nmr_spectra.csv: first column = chemical shift values, each remaining column = one simulated 1H plasma spectrum
- Per-sample annotated relative concentrations of the simulated metabolite mixtures (training labels for the CRNN)

## How to apply

Start from the metabolite reference spectral library shipped with NMRQNet, whose entries store peak signatures (cluster, peak position, width, height) that are rendered as Lorentzian curves on the shared chemical-shift grid defined by NMRQNet/data/tutorial_data/control_human_plasma.csv. For each simulated sample, draw metabolite relative concentrations from a gamma distribution rather than uniform or Gaussian sampling, so that mixture compositions are non-negative and skewed toward a few dominant metabolites, mimicking real plasma concentration behavior. Multiply the rendered Lorentzian signatures by the sampled concentrations and sum them into a mixture spectrum; additionally synthesize the broad lipoprotein and EDTA peak clusters characteristic of plasma. Augment with chemical-shift jitter, multiplet pattern variation, and additive noise before assembly. Write all simulated 1H spectra to synthetic_nmr_spectra.csv with the first column holding chemical shift values and each remaining column holding one simulated sample, matching NMRQNet's required csv input convention.

## Related tools

- **NMRQNet** (Provides the metabolite reference spectral library, the Lorentzian-curve spectrum rendering, and the simulated-data pipeline whose concentration sampling step this skill describes; output csv must match NMRQNet's input convention.) — github.com/LiuzLab/NMRQNet

## Evaluation signals

- synthetic_nmr_spectra.csv exists, parses as csv, has chemical shifts in the first column and exactly one spectral column per simulated sample
- All sampled relative concentrations are non-negative (a property guaranteed by the gamma distribution choice)
- Rendered mixture spectra visually/graphically reconstruct plausible plasma line shapes consistent with the reference-library Lorentzian signatures
- Broad lipoprotein and EDTA peak clusters characteristic of plasma are present in the simulated mixtures
- Downstream: a CRNN trained on these synthetic spectra reconstructs held-out spectra with low residuals and outperforms prior Bayesian or linear-regression-based methods

## Limitations

- Gamma sampling produces synthetic relative concentrations only; the resulting biomarker signals require further experimental confirmation
- The simulation targets 1H plasma spectra; a biofluid-specific analysis pipeline for other biofluid samples was not yet developed
- The overall NMRQNet loss regularizes insufficiently on false-positive metabolite predictions, so final manual visualization checks are still required even for models trained on such synthetic data

## Evidence
