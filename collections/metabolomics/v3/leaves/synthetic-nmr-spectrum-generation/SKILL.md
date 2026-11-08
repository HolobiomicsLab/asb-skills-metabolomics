---
name: synthetic-nmr-spectrum-generation
description: Use when no public annotated NMR spectra database exists and labeled
  training spectra are needed for deep-learning-based metabolite identification/quantification
  in 1H plasma NMR.
when_to_use_negative:
- When experimentally acquired NMR spectra with expert annotations (e.g. Chenomx-fitted
  reports) are already available — simulation is only motivated by the absence of
  a public annotated NMR spectra database
- When the target biofluid is not plasma — the article notes a biofluid-specific analysis
  pipeline for other biofluids was not yet developed, and lipoprotein/EDTA clusters
  are plasma-specific
- When a calibrated spectral deconvolution on a small sample set suffices (e.g. Bayesian
  tools like Batman or ASICS linear-combination fitting) rather than training data
  synthesis for deep learning
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3474
tools:
- name: NMRQNet
  role: Provides the metabolite reference spectral library, the chemical-shift axis
    convention via its tutorial data, and the simulation pipeline used to synthesize
    annotated training spectra for its CRNN model
  repo: github.com/LiuzLab/NMRQNet
- name: Chenomx
  role: Expert manual mapping of metabolite candidates from the reference library
    to mixture spectra — the manual annotation paradigm the synthetic pipeline replaces
    for training-set generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:d86710c826201d091810a6c06a2c664b27c8612886731ec76cdece14956f7e6d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - Chenomx
  - NMRQNet
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# synthetic-nmr-spectrum-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates synthetic annotated 1H NMR plasma spectra for training the NMRQNet CRNN model by rendering metabolite reference-library peak signatures as Lorentzian curves, applying data augmentation (shift jitter, multiplet variation, noise), and mixing concentrations sampled from a gamma distribution.

## When to use

Use when no public annotated NMR spectra database exists and labeled training spectra are needed for deep-learning-based metabolite identification/quantification in 1H plasma NMR.

## When NOT to use

- When experimentally acquired NMR spectra with expert annotations (e.g. Chenomx-fitted reports) are already available — simulation is only motivated by the absence of a public annotated NMR spectra database
- When the target biofluid is not plasma — the article notes a biofluid-specific analysis pipeline for other biofluids was not yet developed, and lipoprotein/EDTA clusters are plasma-specific
- When a calibrated spectral deconvolution on a small sample set suffices (e.g. Bayesian tools like Batman or ASICS linear-combination fitting) rather than training data synthesis for deep learning

## Inputs

- Metabolite reference spectral library packaged with NMRQNet (peak signatures: cluster, peak position, width, height)
- NMRQNet/data/tutorial_data/control_human_plasma.csv (chemical shift axis convention reference)

## Outputs

- synthetic_nmr_spectra.csv: first column = chemical shift values, remaining columns = one simulated 1H plasma spectrum per sample, with annotations (relative concentrations) for CRNN training

## How to apply

Load the metabolite reference spectral library packaged with NMRQNet and read NMRQNet/data/tutorial_data/control_human_plasma.csv to fix the chemical-shift axis convention (first column = shift values, each subsequent column = one sample). Render each reference metabolite signature — defined by its cluster, peak position, width, and height — as a sum of Lorentzian curves on the shared shift grid. Apply data augmentation mimicking environmental and internal spectral variation: random positional (chemical-shift) jitter, multiplet pattern variation, and additive noise. Sample metabolite relative concentrations from a gamma distribution so mixture compositions are realistic and non-negative, then synthesize the broad lipoprotein and EDTA peak clusters characteristic of plasma and add them to the mixture spectra. Assemble the simulated mixtures into synthetic_nmr_spectra.csv in the NMRQNet input format (first column chemical shifts, one spectral column per simulated sample).

## Related tools

- **NMRQNet** (Provides the metabolite reference spectral library, the chemical-shift axis convention via its tutorial data, and the simulation pipeline used to synthesize annotated training spectra for its CRNN model) — github.com/LiuzLab/NMRQNet
- **Chenomx** (Expert manual mapping of metabolite candidates from the reference library to mixture spectra — the manual annotation paradigm the synthetic pipeline replaces for training-set generation)

## Examples

```
pd.read_csv('NMRQNet/data/tutorial_data/control_human_plasma.csv')  # fix shift axis, then render Lorentzian mixtures -> synthetic_nmr_spectra.csv
```

## Evaluation signals

- Output file synthetic_nmr_spectra.csv exists and parses as csv with the first column of chemical shift values and one spectral column per simulated sample
- Rendered mixture spectra visually/graphically reconstruct plausible plasma line shapes consistent with the reference-library Lorentzian signatures
- Sampled relative concentrations are non-negative (gamma-distributed) and yield realistic mixture compositions
- Simulated spectra include the broad lipoprotein and EDTA peak clusters characteristic of plasma
- Augmented variants exhibit positional jitter, multiplet pattern variation, and noise consistent with environmental and internal spectral variation described in the paper

## Limitations

- Synthetic spectra are simulations; model performance must still be validated on real plasma samples (the paper evaluates on mouse-to-human plasma, three anticoagulants, healthy and patient conditions)
- NMRQNet regularizes less on false-positive predictions, so final manual visualization check is required to filter metabolites with false-positive predictions
- The pipeline is plasma-specific: biofluid-specific analysis pipelines for other biofluids were not yet developed

## Evidence

- [methods] The simulation pipeline generates whole NMR spectra for metabolomic candidates by applying Lorentzian-curve functions to the peak signatures: [evidence span withheld — anchor status: not_found]
