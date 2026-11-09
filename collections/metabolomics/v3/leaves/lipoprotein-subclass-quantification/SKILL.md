---
name: lipoprotein-subclass-quantification
description: Use when 1H NMR human plasma spectra (CSV with chemical shifts in the
  first column and one column per sample) need decomposition into metabolite and lipoprotein-cluster
  concentrations via NMRQNet's CRNN initial fitting.
when_to_use_negative:
- Plasma samples collected with heparin or citrate anticoagulants — use NMRQNet's
  other-plasma tutorial (tutorial_for_other_plasma.ipynb) instead of the EDTA pipeline
- The task requires final, locally optimized concentrations — this skill covers only
  the CRNN initial fitting, before global positional correction and local optimization
- Input is not human plasma 1H NMR spectra in NMRQNet's CSV format (e.g., already-quantified
  concentration tables or other biofluids, for which no biofluid-specific pipeline
  is yet developed)
edam_operation: http://edamontology.org/operation_3799
edam_topics: []
tools:
- name: NMRQNet
  role: Deep-learning CRNN pipeline that fits library components (metabolites, 24
    lipoprotein clusters, anticoagulants, DSS) to the preprocessed plasma spectra
    and reconstructs each spectrum
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Notebook prescribing the EDTA-specific free-peak removal and the CRNN fitting
    workflow for EDTA plasma spectra
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:070af2524156a3e4ff935c1eb02b86f667e94481f82ee04292ecd2cef65bf27d
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lipoprotein-subclass-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs NMRQNet's CRNN initial fitting on preprocessed 1H NMR plasma spectra (EDTA free peaks removed) to jointly quantify 65 components, including 24 lipoprotein clusters, 38 metabolites, anticoagulants, and DSS, and reconstruct each spectrum before any global positional correction or local optimization.

## When to use

Use when 1H NMR human plasma spectra (CSV with chemical shifts in the first column and one column per sample) need decomposition into metabolite and lipoprotein-cluster concentrations via NMRQNet's CRNN initial fitting.

## When NOT to use

- Plasma samples collected with heparin or citrate anticoagulants — use NMRQNet's other-plasma tutorial (tutorial_for_other_plasma.ipynb) instead of the EDTA pipeline
- The task requires final, locally optimized concentrations — this skill covers only the CRNN initial fitting, before global positional correction and local optimization
- Input is not human plasma 1H NMR spectra in NMRQNet's CSV format (e.g., already-quantified concentration tables or other biofluids, for which no biofluid-specific pipeline is yet developed)

## Inputs

- CSV of 1H NMR plasma spectra (first column = chemical shift values; each remaining column = one sample's spectrum), NMRQNet input format
- Preprocessed spectra after the NMRQNet preprocessing tutorial and EDTA free-peak removal step
- NMRQNet reference library components (metabolites, lipoprotein clusters, anticoagulants, DSS)

## Outputs

- 65-dimension fitted concentrations per sample (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS), e.g., initial_fitting_results.csv
- Reconstructed spectra for each sample with residuals versus the original spectra
- Flagged metabolites with possible false-positive predictions for final visualization check

## How to apply

Load the plasma spectra CSV with pandas, using the first column as chemical shift values and each remaining column as one sample's 1H NMR spectrum, following the NMRQNet input format exemplified by NMRQNet/data/tutorial_data/control_human_plasma.csv. Preprocess each spectrum per the NMR spectra preprocessing tutorial (https://liuzlab.github.io/NMRQNet/tutorial/data_preprocessing.html), then apply the EDTA-specific free-peak removal step prescribed in NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb. Run the CRNN model with its optimization pipeline on the preprocessed spectra to fit library components and reconstruct each spectrum, producing a 65-dimension output: 38 metabolites, 24 lipoprotein clusters, anticoagulants, and DSS. Save the fitted concentrations and reconstructed spectra (e.g., to initial_fitting_results.csv) and validate that reconstruction closely matches the originals with very low residuals; metabolites in sparse regions (alanine, lactate) should reconstruct most of their clusters, while overlapping-region metabolites (glucose) only reconstruct portions of their clusters, so flag possible false positives for manual visualization check.

## Related tools

- **NMRQNet** (Deep-learning CRNN pipeline that fits library components (metabolites, 24 lipoprotein clusters, anticoagulants, DSS) to the preprocessed plasma spectra and reconstructs each spectrum) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Notebook prescribing the EDTA-specific free-peak removal and the CRNN fitting workflow for EDTA plasma spectra) — github.com/LiuzLab/NMRQNet

## Examples

```
import pandas as pd; df = pd.read_csv('NMRQNet/data/tutorial_data/control_human_plasma.csv')
```

## Evaluation signals

- Reconstruction closely matches the original spectra with very low residuals (the paper's acceptance criterion for accurate reconstruction)
- Output covers all 65 fitted components (38 metabolites + 24 lipoprotein clusters + anticoagulants + DSS) for each of the 40 samples
- Sparse-region metabolites (e.g., alanine, lactate) reconstruct most of their corresponding clusters; overlapping-region metabolites (e.g., glucose) reconstruct only portions — deviations from this pattern indicate fitting problems
- Manual visualization check confirms flagged components are not false-positive predictions (NMRQNet regularizes insufficiently on false positives)
- Quantification consistency across samples, as scored by RMSE, Pearson correlation, and residuals

## Limitations

- NMRQNet regularizes less on false-positive predictions because its loss focuses on eliminating residuals, so a final manual visualization check is required and no automated pre-filtering or post-processing step exists
- Metabolites in overlapping spectral regions (e.g., glucose) can only reconstruct portions of their clusters during initial fitting
- The biofluid-specific analysis pipeline is not yet developed for biofluids other than plasma, and proposed biomarkers require further experimental confirmation

## Evidence
