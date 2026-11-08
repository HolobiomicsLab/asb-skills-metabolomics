---
name: nmr-spectral-data-augmentation
description: Use when training deep-learning metabolite identification/quantification
  models (e.g. CRNN in NMRQNet) but no public annotated NMR spectra database exists
  and training spectra must be synthesized from a metabolite reference spectral library.
when_to_use_negative:
- A public annotated NMR spectra database with sufficient real labeled training samples
  already exists for your target biofluid.
- You need to analyze a biofluid other than plasma — the article notes the biofluid-specific
  analysis pipeline is future work, and the simulation here models plasma-specific
  lipoprotein and EDTA clusters.
- The task is quantification of real patient samples rather than training-data synthesis;
  simulated gamma-distributed concentrations are not measurements of actual samples.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3520
tools:
- name: NMRQNet
  role: Provides the metabolite reference spectral library, the Lorentzian-curve rendering
    of peak signatures, and the augmentation pipeline used to synthesize the training
    spectra.
  repo: github.com/LiuzLab/NMRQNet
- name: Chenomx
  role: 'Reference manual curation workflow that the automated simulation replaces:
    NMR experts manually map metabolite candidates to the mixture and generate a fitting
    report.'
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:ca83ac859ac5f38ff5d58ad4c8f0f01a1b7daa2747944470387fd5d63d8e8052
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# nmr-spectral-data-augmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates synthetic annotated 1H NMR plasma spectra by rendering metabolite reference-library peak signatures (cluster, position, width, height) as Lorentzian curves and augmenting them with positional/pattern jitter, gamma-distributed concentrations, and additive noise, so CRNN deep-learning models can be trained without a public annotated NMR database.

## When to use

Use when training deep-learning metabolite identification/quantification models (e.g. CRNN in NMRQNet) but no public annotated NMR spectra database exists and training spectra must be synthesized from a metabolite reference spectral library.

## When NOT to use

- A public annotated NMR spectra database with sufficient real labeled training samples already exists for your target biofluid.
- You need to analyze a biofluid other than plasma — the article notes the biofluid-specific analysis pipeline is future work, and the simulation here models plasma-specific lipoprotein and EDTA clusters.
- The task is quantification of real patient samples rather than training-data synthesis; simulated gamma-distributed concentrations are not measurements of actual samples.

## Inputs

- Metabolite reference spectral library packaged with NMRQNet (peak clusters, peak positions, widths, heights per metabolite)
- Tutorial example spectrum NMRQNet/data/tutorial_data/control_human_plasma.csv for chemical-shift axis convention
- Target metabolite set (e.g. the nine-metabolite reference set used for CRNN identification training)

## Outputs

- synthetic_nmr_spectra.csv: csv with first column = chemical shift values, each subsequent column = one simulated 1H plasma spectrum
- Simulated mixtures including broad lipoprotein and EDTA peak clusters characteristic of plasma
- Annotated training/test spectra with metabolite presence/absence labels and relative concentrations for CRNN training

## How to apply

Load the metabolite reference spectral library shipped with NMRQNet and read NMRQNet/data/tutorial_data/control_human_plasma.csv to fix the chemical-shift axis convention (first column = shift values, each subsequent column = one sample). Render each reference metabolite signature — its peak cluster, position, width and height — as a sum of Lorentzian curves on the shared shift grid. Apply data augmentation to the rendered signatures with random chemical-shift (positional) jitter, multiplet pattern variation, and additive noise to mimic environmental and internal spectral variation, and sample metabolite relative concentrations from a gamma distribution to build realistic non-negative mixtures. Add the broad lipoprotein and EDTA peak clusters characteristic of plasma to the mixture spectra, then assemble the simulated mixtures into a csv (synthetic_nmr_spectra.csv) whose first column holds chemical shift values and whose remaining columns each hold one simulated 1H plasma spectrum. The augmentation is what overcomes the data shortage issue for deep learning in metabolomics, since fixed-position rendering would not generalize to real spectra with positional and pattern variations.

## Related tools

- **NMRQNet** (Provides the metabolite reference spectral library, the Lorentzian-curve rendering of peak signatures, and the augmentation pipeline used to synthesize the training spectra.) — github.com/LiuzLab/NMRQNet
- **Chenomx** (Reference manual curation workflow that the automated simulation replaces: NMR experts manually map metabolite candidates to the mixture and generate a fitting report.)

## Examples

```
jupyter notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- The output csv parses with the first column of chemical shift values and exactly one spectral column per simulated sample, matching the NMRQNet input format.
- Rendered mixture spectra visually reconstruct plausible plasma line shapes consistent with the reference-library Lorentzian signatures.
- Held-out simulated test samples show expected identification accuracy: 92% overall presence/absence accuracy and per-metabolite average accuracies in the 0.978–0.995 range (Supplementary Tables 1–2).
- Augmented spectra differ from the fixed reference positions via positional/pattern variation, confirming the model generalizes beyond fixed peak positions.
- Simulated concentrations follow the gamma distribution used for sampling, i.e. non-negative relative-concentration mixtures.

## Limitations

- NMRQNet regularizes insufficiently on false-positive predictions — the residual-focusing loss does not penalize spurious metabolite presence, requiring a final manual visualization check to filter false positives.
- Further experiments are needed to confirm the potential biomarkers proposed from simulated-data-derived concentration estimates.
- Only plasma (with EDTA, heparin, citrate anticoagulants) is covered; a biofluid-specific analysis pipeline for other biofluids is future work.

## Evidence

- [other] Lorentzian rendering and augmentation design: [evidence span withheld — anchor status: not_found]
- [results] Identification accuracy benchmark: [evidence span withheld — anchor status: not_found]
