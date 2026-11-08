---
name: lipoprotein-edta-cluster-simulation
description: Use when no public annotated NMR spectra database exists and synthetic
  labeled 1H plasma training spectra with realistic lipoprotein/EDTA clusters are
  needed to train a quantification model such as CRNN.
when_to_use_negative:
- The input is real experimental 1H NMR spectra to be quantified — this skill generates
  synthetic training data, not an inference or fitting pipeline
- The target biofluid is not human plasma — the simulated lipoprotein and EDTA clusters
  are characteristic of plasma and the paper notes biofluid-specific pipelines are
  not yet developed
- Raw FID/free-induction-decay data still needs preprocessing before spectral analysis
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: NMRQNet
  role: Provides the metabolite reference spectral library (peak cluster, position,
    width, height entries) and the tutorial example spectrum control_human_plasma.csv
    used to render Lorentzian-based simulated plasma spectra with lipoprotein and
    EDTA clusters
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:42ff44b347de61d2af29adf0f1cf5772aad07fb816254bae540072866c680973
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
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lipoprotein-edta-cluster-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Synthesizes annotated training spectra for NMR metabolite quantification by rendering metabolite reference signatures as Lorentzian curves and layering the broad lipoprotein and EDTA peak clusters characteristic of plasma, replacing the missing public annotated NMR spectra database.

## When to use

Use when no public annotated NMR spectra database exists and synthetic labeled 1H plasma training spectra with realistic lipoprotein/EDTA clusters are needed to train a quantification model such as CRNN.

## When NOT to use

- The input is real experimental 1H NMR spectra to be quantified — this skill generates synthetic training data, not an inference or fitting pipeline
- The target biofluid is not human plasma — the simulated lipoprotein and EDTA clusters are characteristic of plasma and the paper notes biofluid-specific pipelines are not yet developed
- Raw FID/free-induction-decay data still needs preprocessing before spectral analysis

## Inputs

- NMRQNet metabolite reference spectral library (peak cluster, peak position, width, height per metabolite)
- NMRQNet/data/tutorial_data/control_human_plasma.csv to fix the chemical shift axis convention
- Gamma-distributed samples of metabolite relative concentrations for mixture compositions

## Outputs

- synthetic_nmr_spectra.csv — csv whose first column is chemical shift values and whose remaining columns each hold one simulated 1H plasma spectrum with metabolite mixtures plus lipoprotein and EDTA clusters
- Annotated synthetic training spectra (metabolite identities/relative concentrations) for CRNN training

## How to apply

Load the metabolite reference spectral library packaged with NMRQNet, whose entries store peak cluster, peak position, width, and height, and read NMRQNet/data/tutorial_data/control_human_plasma.csv to fix the chemical-shift axis convention (first column = shift values, each subsequent column = one sample). Render each reference metabolite signature as a sum of Lorentzian curves on the shared shift grid, then apply data augmentation mimicking 'positional and pattern variations brought by environmental and internal factors': random chemical-shift jitter, multiplet pattern variation, and additive noise. Sample metabolite relative concentrations from a gamma distribution so mixture compositions are realistic and non-negative. Synthesize the broad lipoprotein and EDTA peak clusters characteristic of plasma and add them to each mixture spectrum. Finally, assemble the simulated mixtures into a csv file (synthetic_nmr_spectra.csv) whose first column holds chemical shift values and each remaining column holds one simulated 1H plasma spectrum.

## Related tools

- **NMRQNet** (Provides the metabolite reference spectral library (peak cluster, position, width, height entries) and the tutorial example spectrum control_human_plasma.csv used to render Lorentzian-based simulated plasma spectra with lipoprotein and EDTA clusters) — github.com/LiuzLab/NMRQNet

## Examples

```
jupyter notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- synthetic_nmr_spectra.csv exists and parses as csv with the first column of chemical shift values and one spectral column per simulated sample
- Rendered mixture spectra visually/graphically reconstruct plausible plasma line shapes consistent with the reference-library Lorentzian signatures
- Broad lipoprotein and EDTA peak clusters characteristic of plasma are present in each simulated spectrum
- Sampled metabolite relative concentrations are non-negative and gamma-distributed, giving realistic mixture compositions
- Simulated spectra carry the intended augmentation variation (shift jitter, multiplet pattern variation, additive noise) so a model trained on them tolerates positional and pattern variation

## Limitations

- The simulation is plasma-specific: the broad lipoprotein and EDTA clusters mimic plasma and EDTA anticoagulant, and the paper notes 'Future efforts are needed to develop the biofluid-specific analysis pipeline'
- The pipeline overcame data shortage only by simulation — the synthetic spectra are approximations of environmental and internal spectral variation, not measured annotated spectra
- Covers the 39 commonly measurable metabolites suggested by the IVDr (In Vitro Diagnostic Research) platform from Bruker, not an exhaustive metabolite set

## Evidence

- [other] The simulation pipeline generates whole NMR spectra for metabolomic candidates by applying Lorentzian-curve functions to the peak signatures: [evidence span withheld — anchor status: not_found]
- [other] Synthesize broad lipoprotein and EDTA peak clusters characteristic of plasma and add them to the mixture spectra: [evidence span withheld — anchor status: not_found]
