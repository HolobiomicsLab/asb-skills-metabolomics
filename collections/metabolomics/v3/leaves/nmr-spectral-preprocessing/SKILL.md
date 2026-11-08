---
name: nmr-spectral-preprocessing
description: Use when raw 1H NMR spectra (e.g. EDTA, heparin, or citrate anticoagulated
  plasma) must be cleaned and formatted into the NMRQNet CSV layout prior to CRNN
  identification or quantification.
when_to_use_negative:
- Input is already a preprocessed NMRQNet-format CSV with anticoagulant peaks removed
  — re-running preprocessing risks corrupting the spectra.
- The analysis target is not 1H NMR spectra (e.g. mass spectrometry or chromatography
  data) — the anticoagulant-specific cleaning and CSV layout are NMR-specific.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: NMRQNet
  role: Provides the data preprocessing tutorial and the CSV input format the preprocessed
    spectra must conform to before CRNN identification and quantification.
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Notebook implementing the EDTA-specific free-peak removal step for EDTA-anticoagulated
    plasma spectra.
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:009fc9d04b1ac717f5ac7fde1f9bc0c0283ecba396e0bf39cc7fe7ace5da2716
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nmr-spectral-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare raw 1H NMR plasma spectra into the NMRQNet CSV input format — chemical shift values in the first column, one sample spectrum per subsequent column — following NMRQNet's data preprocessing tutorial, including anticoagulant-specific handling such as EDTA free-peak removal before downstream CRNN identification and quantification.

## When to use

Use when raw 1H NMR spectra (e.g. EDTA, heparin, or citrate anticoagulated plasma) must be cleaned and formatted into the NMRQNet CSV layout prior to CRNN identification or quantification.

## When NOT to use

- Input is already a preprocessed NMRQNet-format CSV with anticoagulant peaks removed — re-running preprocessing risks corrupting the spectra.
- The analysis target is not 1H NMR spectra (e.g. mass spectrometry or chromatography data) — the anticoagulant-specific cleaning and CSV layout are NMR-specific.

## Inputs

- Raw 1H NMR spectra of plasma samples (one spectrum per sample) with chemical shift axis
- Anticoagulant type annotation (EDTA vs heparin vs citrate) to select the correct tutorial route

## Outputs

- Preprocessed NMRQNet-format CSV: first column chemical shift values, one column per sample spectrum
- Spectra with anticoagulant free peaks removed (e.g. EDTA peaks removed for EDTA plasma)

## How to apply

Read the raw spectra with pandas, using the first column as chemical shift values and each subsequent column as one plasma sample's 1H NMR spectrum, matching the layout of NMRQNet/data/tutorial_data/control_human_plasma.csv. Follow NMRQNet's data preprocessing tutorial (https://liuzlab.github.io/NMRQNet/tutorial/data_preprocessing.html) to clean the spectra. Because anticoagulants produce their own free peaks, choose the preprocessing route by anticoagulant: for EDTA plasma apply the EDTA-specific free-peak removal step from the tutorial_for_EDTA_plasma.ipynb notebook, whereas heparin and citrate plasma follow tutorial_for_other_plasma.ipynb. The output is a preprocessed CSV ready for the CRNN model and NMRQNet's optimization pipeline, which is essential because NMRQNet reconstructs the whole spectrum and residual artifacts from unremoved anticoagulant peaks would degrade the reconstruction.

## Related tools

- **NMRQNet** (Provides the data preprocessing tutorial and the CSV input format the preprocessed spectra must conform to before CRNN identification and quantification.) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Notebook implementing the EDTA-specific free-peak removal step for EDTA-anticoagulated plasma spectra.) — github.com/LiuzLab/NMRQNet

## Examples

```
import pandas as pd; df = pd.read_csv('NMRQNet/data/tutorial_data/control_human_plasma.csv'); shifts = df.iloc[:, 0]; spectra = df.iloc[:, 1:]
```

## Evaluation signals

- The preprocessed CSV's first column contains chemical shift values and every subsequent column is a single sample spectrum, matching the layout of NMRQNet/data/tutorial_data/control_human_plasma.csv.
- No anticoagulant free peaks remain in the preprocessed spectra (e.g. EDTA peaks absent for EDTA plasma samples).
- The correct anticoagulant-specific tutorial route was followed (tutorial_for_EDTA_plasma.ipynb for EDTA; tutorial_for_other_plasma.ipynb for heparin/citrate).
- Downstream CRNN fitting on the preprocessed spectra can reconstruct the original spectra with very low residuals, indicating preprocessing did not distort the metabolite signal.

## Limitations

- NMRQNet regularizes less on false-positive predictions, so a final manual visualization check is still required to filter metabolites with false-positive predictions even after correct preprocessing.
- Preprocessing routes are anticoagulant-specific; applying the EDTA notebook to heparin or citrate plasma (or vice versa) would leave or remove the wrong free peaks.
- The preprocessing pipeline is biofluid-specific to plasma; other biofluids need a dedicated analysis pipeline that does not yet exist.

## Evidence
