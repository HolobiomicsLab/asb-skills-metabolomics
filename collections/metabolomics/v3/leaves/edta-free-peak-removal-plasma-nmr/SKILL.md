---
name: edta-free-peak-removal-plasma-nmr
description: Use when input spectra are 1H NMR spectra of EDTA-anticoagulated human
  plasma (e.g., the 40 MDS-family plasma samples) and the goal is accurate metabolite/lipoprotein
  quantification with NMRQNet before CRNN fitting.
when_to_use_negative:
- Plasma anticoagulated with heparin or citrate instead of EDTA — NMRQNet provides
  a separate tutorial (tutorial_for_other_plasma.ipynb) for those samples
- Spectra that have already undergone EDTA free-peak removal and preprocessing in
  a prior run
- Non-plasma biofluids — the article notes the biofluid-specific analysis pipeline
  was not yet developed, so this plasma-specific step does not generalize
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3334
tools:
- name: NMRQNet
  role: Provides the input CSV format, the data preprocessing tutorial, and the EDTA-specific
    plasma analysis notebook that implements the free-peak removal before CRNN fitting
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: The prescribed notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb)
    carrying out the EDTA-specific free-peak removal step on EDTA plasma spectra
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:cd8f93d78bb6cdea0d7522f65c68e169242a443c9f22f91040c862ed2eb4fcbb
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# edta-free-peak-removal-plasma-nmr

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A preprocessing step in the NMRQNet pipeline that removes EDTA-anticoagulant free peaks from EDTA-plasma 1H NMR spectra before CRNN component fitting, so that the 65-component quantification (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) is not contaminated by the anticoagulant's own spectral features.

## When to use

Use when input spectra are 1H NMR spectra of EDTA-anticoagulated human plasma (e.g., the 40 MDS-family plasma samples) and the goal is accurate metabolite/lipoprotein quantification with NMRQNet before CRNN fitting.

## When NOT to use

- Plasma anticoagulated with heparin or citrate instead of EDTA — NMRQNet provides a separate tutorial (tutorial_for_other_plasma.ipynb) for those samples
- Spectra that have already undergone EDTA free-peak removal and preprocessing in a prior run
- Non-plasma biofluids — the article notes the biofluid-specific analysis pipeline was not yet developed, so this plasma-specific step does not generalize

## Inputs

- CSV file of EDTA-plasma 1H NMR spectra (first column chemical shifts; one column per plasma sample), NMRQNet input format e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv
- EDTA-anticoagulated human plasma samples (e.g., the 40 MDS-family plasma samples)

## Outputs

- Preprocessed spectra with EDTA-specific free peaks removed, ready for CRNN library-component fitting
- Downstream CRNN outputs: 65-dimension per-sample concentrations (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) and reconstructed spectra

## How to apply

Load the plasma spectra CSV (first column = chemical shift values, each remaining column = one sample's 1H NMR spectrum, per the NMRQNet input format exemplified by control_human_plasma.csv). First preprocess each spectrum following the NMR spectra preprocessing tutorial at https://liuzlab.github.io/NMRQNet/tutorial/data_preprocessing.html. Then apply the EDTA-specific free-peak removal procedure exactly as prescribed in the EDTA plasma notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb — this is a distinct tutorial path because anticoagulant identity changes the artifact peaks present in the mixture. The rationale is that EDTA contributes its own free peaks to the spectrum; leaving them in place would bias the CRNN reconstruction and concentration estimates for the 65 fitted components. Only after this removal step should the CRNN model with its optimization pipeline be run to fit library components; the corrected spectra then feed the initial fitting, and downstream validation checks reconstruction residuals and possible false-positive metabolite predictions.

## Related tools

- **NMRQNet** (Provides the input CSV format, the data preprocessing tutorial, and the EDTA-specific plasma analysis notebook that implements the free-peak removal before CRNN fitting) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (The prescribed notebook (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) carrying out the EDTA-specific free-peak removal step on EDTA plasma spectra) — github.com/LiuzLab/NMRQNet

## Evaluation signals

- After the full CRNN fitting on the EDTA-peak-removed spectra, the reconstruction should closely match the original spectra with very low residuals (the paper's reported acceptance criterion)
- The downstream output must cover all 65 fitted components (38 metabolites, 24 lipoprotein clusters, anticoagulants, DSS) for each of the 40 samples
- Metabolites with possible false-positive predictions should be flagged for the final manual visualization check
- EDTA free-peak artifacts should no longer drive the fit: fitted anticoagulant/DSS components should reflect the corrected spectra, not residual EDTA peaks

## Limitations

- NMRQNet regularizes insufficiently on false-positive predictions because its loss focuses on eliminating residuals between original and reconstructed spectra; a final visualization check is required, and no automated pre-filtering or post-processing step exists
- The EDTA procedure is anticoagulant-specific; heparin and citrate plasma require the separate tutorial_for_other_plasma.ipynb path
- A biofluid-specific analysis pipeline for other biofluid samples was not yet developed

## Evidence
