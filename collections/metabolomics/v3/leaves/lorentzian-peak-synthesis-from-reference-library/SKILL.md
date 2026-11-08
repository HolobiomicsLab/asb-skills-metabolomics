---
name: lorentzian-peak-synthesis-from-reference-library
description: Use when training an NMR metabolite identification/quantification model
  (e.g., CRNN) but no annotated spectra database is available, and synthetic 1H plasma
  training spectra with known metabolite concentrations must be simulated.
when_to_use_negative:
- The input is real measured plasma NMR spectra that need metabolite identification/quantification
  — that is NMRQNet's inference task, not training-data synthesis
- The target biofluid is not plasma — the article notes a biofluid-specific analysis
  pipeline was not yet developed, and the lipoprotein/EDTA clusters are plasma- and
  anticoagulant-specific
- A public annotated NMR spectra database exists for the metabolites of interest,
  removing the data-shortage motivation for simulation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NMRQNet
  role: Provides the metabolite reference spectral library and the simulation/augmentation
    pipeline used to render Lorentzian-based synthetic spectra for training the CRNN
    model
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:388300b79dee01b679cf2dd722eebe2f2ed1171889a5f70229e9bbf9bf6ce59d
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lorentzian-peak-synthesis-from-reference-library

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates synthetic annotated 1H NMR plasma spectra by rendering metabolite reference-library peak signatures (cluster, peak position, width, height) as sums of Lorentzian curves, then augmenting them to train deep-learning quantification models like NMRQNet when no public annotated NMR database exists.

## When to use

Use when training an NMR metabolite identification/quantification model (e.g., CRNN) but no annotated spectra database is available, and synthetic 1H plasma training spectra with known metabolite concentrations must be simulated.

## When NOT to use

- The input is real measured plasma NMR spectra that need metabolite identification/quantification — that is NMRQNet's inference task, not training-data synthesis
- The target biofluid is not plasma — the article notes a biofluid-specific analysis pipeline was not yet developed, and the lipoprotein/EDTA clusters are plasma- and anticoagulant-specific
- A public annotated NMR spectra database exists for the metabolites of interest, removing the data-shortage motivation for simulation

## Inputs

- Metabolite reference spectral library packaged with NMRQNet (peak signatures: cluster, peak position, width, height)
- Tutorial example spectrum NMRQNet/data/tutorial_data/control_human_plasma.csv to fix chemical shift axis convention

## Outputs

- synthetic_nmr_spectra.csv — first column of chemical shift values, remaining columns each holding one simulated 1H plasma spectrum with associated simulated metabolite relative concentrations (annotations)

## How to apply

Load the metabolite reference spectral library packaged with NMRQNet and fix the chemical shift axis convention using the tutorial example NMRQNet/data/tutorial_data/control_human_plasma.csv (first column = shift values, each subsequent column = one sample). Render each reference metabolite signature as a sum of Lorentzian curves on the shared shift grid, applying data augmentation — random chemical-shift (positional) jitter, multiplet pattern variation, and additive noise — to mimic environmental and internal spectral variation. Sample metabolite relative concentrations from a gamma distribution to build realistic non-negative mixture compositions, then synthesize the broad lipoprotein and EDTA peak clusters characteristic of plasma and add them to the mixture spectra. Assemble the mixtures into synthetic_nmr_spectra.csv and validate that it parses with the expected shift/spectrum column layout and that rendered line shapes resemble the reference Lorentzian signatures.

## Related tools

- **NMRQNet** (Provides the metabolite reference spectral library and the simulation/augmentation pipeline used to render Lorentzian-based synthetic spectra for training the CRNN model) — github.com/LiuzLab/NMRQNet

## Evaluation signals

- Output file synthetic_nmr_spectra.csv exists and parses as csv with the first column of chemical shifts and one spectral column per simulated sample
- Rendered mixture spectra visually/graphically reconstruct plausible plasma line shapes consistent with the reference-library Lorentzian signatures
- Sampled metabolite relative concentrations are non-negative and drawn from the gamma distribution (realistic mixture compositions)
- Simulated spectra contain the broad lipoprotein and EDTA peak clusters characteristic of plasma
- Augmented spectra exhibit positional (chemical-shift) jitter, multiplet pattern variation, and additive noise as intended

## Limitations

- The simulation is tuned to 1H plasma spectra (including EDTA and lipoprotein clusters); other biofluids or anticoagulants (heparin, citrate) are not covered by this synthesis design
- This pipeline addresses training-data scarcity but does not by itself fix downstream model limitations — NMRQNet regularized less on false-positive predictions, still requiring a final manual visualization check
- Embedded variation is limited to positional jitter, multiplet pattern variation, and additive noise; other environmental/internal spectral effects may be underrepresented

## Evidence

- [full_text] Simulation pipeline core: [evidence span withheld — anchor status: not_found]
