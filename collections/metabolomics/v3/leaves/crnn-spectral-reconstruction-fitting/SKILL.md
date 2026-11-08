---
name: crnn-spectral-reconstruction-fitting
description: Use when running the first-pass CRNN fit of NMRQNet on preprocessed EDTA-plasma
  1H NMR spectra (CSV of chemical shifts + per-sample spectra) to obtain a 65-dimension
  component quantification and check which metabolites reconstruct their spectral
  clusters.
when_to_use_negative:
- Input spectra are heparin or citrate plasma rather than EDTA plasma — use NMRQNet/tutorial/tutorial_for_other_plasma.ipynb
  instead of the EDTA free-peak-removal step
- You need the final quantification after global positional correction and local optimization,
  not the initial CRNN fitting snapshot
- Input is not a per-sample 1H NMR plasma spectrum CSV (e.g. a feature table or other
  biofluid data — the biofluid-specific pipeline is not yet developed)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRQNet
  role: Deep-learning CRNN + optimization pipeline that fits library components and
    reconstructs the whole 1H NMR spectrum to quantify metabolites
  repo: https://github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Notebook prescribing preprocessing and the EDTA-specific free-peak removal
    step before CRNN fitting
  repo: https://github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:1b86b9a7c993782136a93c53dfe786cf1ef08f21383cbb4e05afc602216a7d80
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  repo_url: https://github.com/LiuzLab/NMRQNet
  tools:
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# crnn-spectral-reconstruction-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Initial spectral reconstruction and quantification fitting of 65 plasma components (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) using NMRQNet's CRNN model on preprocessed 1H NMR spectra, before global positional correction and local optimization.

## When to use

Use when running the first-pass CRNN fit of NMRQNet on preprocessed EDTA-plasma 1H NMR spectra (CSV of chemical shifts + per-sample spectra) to obtain a 65-dimension component quantification and check which metabolites reconstruct their spectral clusters.

## When NOT to use

- Input spectra are heparin or citrate plasma rather than EDTA plasma — use NMRQNet/tutorial/tutorial_for_other_plasma.ipynb instead of the EDTA free-peak-removal step
- You need the final quantification after global positional correction and local optimization, not the initial CRNN fitting snapshot
- Input is not a per-sample 1H NMR plasma spectrum CSV (e.g. a feature table or other biofluid data — the biofluid-specific pipeline is not yet developed)

## Inputs

- MDS-family plasma 1H NMR spectra CSV (first column = chemical shift values, remaining columns = one spectrum per sample; NMRQNet input format)
- NMR spectra preprocessing tutorial (https://liuzlab.github.io/NMRQNet/tutorial/data_preprocessing.html)
- EDTA plasma analysis tutorial notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb)

## Outputs

- 65-dimension fitted component concentrations per sample (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS)
- Reconstructed spectra for all 40 samples (e.g. initial_fitting_results.csv)
- Residuals between original and reconstructed spectra

## How to apply

Load the MDS-family plasma spectra CSV with pandas using the first column as chemical shift values and each remaining column as one sample's 1H NMR spectrum, following the NMRQNet input format (NMRQNet/data/tutorial_data/control_human_plasma.csv is the format example). Preprocess each spectrum per the NMR spectra preprocessing tutorial (https://liuzlab.github.io/NMRQNet/tutorial/data_preprocessing.html), then apply the EDTA-specific free-peak removal step from NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb. Run the CRNN model with its optimization pipeline on the 40 preprocessed spectra to fit library components and reconstruct each spectrum, producing a 65-dimension output per sample: 38 metabolites, 24 lipoprotein clusters, anticoagulants, and DSS. Save fitted concentrations and reconstructed spectra for all 40 samples (e.g. to initial_fitting_results.csv). Rationale: at this initial-fitting stage, metabolites in sparse spectral regions (alanine, lactate) reconstruct most of their clusters, while metabolites in overlapping regions (glucose) reconstruct only portions — global positional correction and local optimization come later.

## Related tools

- **NMRQNet** (Deep-learning CRNN + optimization pipeline that fits library components and reconstructs the whole 1H NMR spectrum to quantify metabolites) — https://github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Notebook prescribing preprocessing and the EDTA-specific free-peak removal step before CRNN fitting) — https://github.com/LiuzLab/NMRQNet

## Examples

```
import pandas as pd; df = pd.read_csv('mds_plasma_spectra.csv'); spectra = df.iloc[:, 1:]  # then run NMRQNet per tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- Reconstruction closely matches the original spectra with very low residuals (the paper's acceptance criterion for accurate reconstruction)
- Output covers all 65 fitted components (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) for each of the 40 samples
- Sparse-region metabolites (e.g. alanine, lactate) reconstruct most of their corresponding clusters in the initial fitting
- Overlapping-region metabolites (e.g. glucose) are flagged as only partially reconstructed, pending later correction stages
- Metabolites with possible false-positive predictions are flagged for the final manual visualization check

## Limitations

- NMRQNet regularizes less on false-positive predictions because the loss focuses on eliminating residuals between original and reconstructed spectra; a final visualization check is required to filter false-positive metabolites
- In the initial fitting (before global positional correction and local optimization), metabolites in overlapping regions such as glucose reconstruct only portions of their clusters
- No automated pre-filtering or post-processing step exists for false-positive predictions; the manual check is currently required

## Evidence
