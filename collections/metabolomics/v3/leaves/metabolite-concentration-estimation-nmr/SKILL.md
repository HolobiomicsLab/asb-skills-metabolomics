---
name: metabolite-concentration-estimation-nmr
description: Use when you have preprocessed 1H NMR plasma spectra (e.g., CSV with
  chemical shifts in the first column, one column per sample) and need automatic identification
  and quantification of dominant metabolites without manual library mapping.
when_to_use_negative:
- Samples are other biofluids (urine, CSF, etc.) — the article states a biofluid-specific
  analysis pipeline beyond plasma is not yet developed
- The anticoagulant is heparin or citrate rather than EDTA — use tutorial_for_other_plasma.ipynb
  workflow instead of the EDTA free-peak removal step
- You need fully automated false-positive filtering — NMRQNet currently requires a
  final manual visualization check, so a fully hands-off pipeline is not achievable
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: NMRQNet
  role: Deep-learning CRNN (CNN + GRU) pipeline with an optimization loop that fits
    library components to preprocessed plasma 1H NMR spectra and outputs metabolite/lipoprotein
    concentrations and reconstructed spectra
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Notebook tutorial prescribing preprocessing and EDTA-specific free-peak removal
    before running the CRNN quantification on EDTA plasma spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:fdef28cf7351db085753dbdfc9356ceb51f345b3255ce7f3e215a2e5275d579f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolite-concentration-estimation-nmr

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate relative concentrations of 65 plasma components (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) from 1H NMR spectra using NMRQNet's CRNN (CNN + GRU) model with its optimization pipeline, producing fitted concentrations and reconstructed spectra with very low residuals.

## When to use

Use when you have preprocessed 1H NMR plasma spectra (e.g., CSV with chemical shifts in the first column, one column per sample) and need automatic identification and quantification of dominant metabolites without manual library mapping.

## When NOT to use

- Samples are other biofluids (urine, CSF, etc.) — the article states a biofluid-specific analysis pipeline beyond plasma is not yet developed
- The anticoagulant is heparin or citrate rather than EDTA — use tutorial_for_other_plasma.ipynb workflow instead of the EDTA free-peak removal step
- You need fully automated false-positive filtering — NMRQNet currently requires a final manual visualization check, so a fully hands-off pipeline is not achievable

## Inputs

- CSV of 1H NMR spectra: first column chemical shift values, one column per plasma sample (NMRQNet input format, e.g., control_human_plasma.csv)
- Preprocessed spectra after the NMR spectra preprocessing tutorial steps
- For EDTA plasma: EDTA free-peak-removed spectra per tutorial_for_EDTA_plasma.ipynb
- Reference metabolite/lipoprotein library components (65 components: 38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS)

## Outputs

- 65-dimension fitted concentration vector per sample (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS)
- Reconstructed spectrum per sample with residuals vs. the original spectrum
- Results table such as initial_fitting_results.csv with concentrations and reconstructed spectra for all samples
- List of metabolites flagged as possible false positives for manual visualization check

## How to apply

Format input as a CSV where the first column holds chemical shift values and each remaining column is one sample's 1H NMR spectrum, following the NMRQNet format (see NMRQNet/data/tutorial_data/control_human_plasma.csv). Preprocess each spectrum per the NMR spectra preprocessing tutorial; for EDTA-anticoagulated plasma, apply the EDTA-specific free-peak removal step from NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb (for heparin or citrate plasma use tutorial_for_other_plasma.ipynb instead). Run the CRNN model with its optimization pipeline on the preprocessed spectra to fit library components and reconstruct each spectrum, yielding a 65-dimension output per sample. For EDTA plasma such as the 40 MDS-family samples, an initial CRNN fitting before global positional correction and local optimization reconstructs most clusters for metabolites in sparse regions (e.g., alanine, lactate) but only portions of clusters for metabolites in overlapping regions (e.g., glucose); save fitted concentrations and reconstructed spectra (e.g., initial_fitting_results.csv). Because NMRQNet regularizes less on false-positive predictions, finish with a manual visualization check to filter spurious metabolite assignments.

## Related tools

- **NMRQNet** (Deep-learning CRNN (CNN + GRU) pipeline with an optimization loop that fits library components to preprocessed plasma 1H NMR spectra and outputs metabolite/lipoprotein concentrations and reconstructed spectra) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Notebook tutorial prescribing preprocessing and EDTA-specific free-peak removal before running the CRNN quantification on EDTA plasma spectra)

## Examples

```
pd.read_csv('NMRQNet/data/tutorial_data/control_human_plasma.csv'); # then run NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb on the preprocessed spectra
```

## Evaluation signals

- Reconstructed spectrum closely matches the original spectrum with very low residuals — the paper's acceptance criterion
- Output covers all 65 fitted components (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) for each sample
- Evaluation metrics RMSE, Pearson correlation, and residuals are improved and consistent among samples compared to Bayesian (Batman) or linear-regression (ASICS) baselines
- Flag metabolites with possible false-positive predictions for the final manual visualization check
- For an initial (pre-correction) CRNN fit, expect sparse-region metabolites (alanine, lactate) to reconstruct most of their clusters while overlapping-region metabolites (glucose) reconstruct only portions

## Limitations

- NMRQNet regularizes insufficiently on false-positive predictions; a final manual visualization check is required to filter them
- Further experiments were needed to confirm the proposed potential biomarkers
- A biofluid-specific analysis pipeline for other biofluid samples is not yet developed; current scope is plasma
- In overlapping spectral regions, metabolites like glucose may only partially reconstruct their clusters, especially in the initial fitting before global positional correction and local optimization

## Evidence
