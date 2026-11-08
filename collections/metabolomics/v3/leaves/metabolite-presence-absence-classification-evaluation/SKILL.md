---
name: metabolite-presence-absence-classification-evaluation
description: Use when a trained NMRQNet CRNN model must be scored on held-out simulated
  nine-metabolite 1H NMR spectra for presence/absence identification accuracy.
when_to_use_negative:
- The task is quantification of relative concentrations or spectral reconstruction
  rather than presence/absence identification — the quantification pipeline and its
  RMSE/Pearson/residual metrics apply instead
- The spectra are real experimental plasma samples rather than the simulated nine-metabolite
  test split, since accuracy benchmarks reported here are defined on simulated data
- A single metabolite's signal must be inspected manually for false-positive calls
  — NMRQNet's false positives require a final visualization check rather than this
  aggregate accuracy metric
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_3474
tools:
- name: NMRQNet
  role: Provides the data-augmentation simulation pipeline for generating nine-metabolite
    1H NMR training/test splits, the csv input format, the CRNN (CNN + GRU) identification
    model and optimization pipeline, and the accuracy evaluation
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:f67bc2fbe9c2d926b7bbdd1151ebd9683f6028b80adf1e46dd3ca62f826795d4
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
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolite-presence-absence-classification-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates how well a trained CRNN (CNN + GRU) identification model from NMRQNet predicts the presence or absence of each of nine metabolites on held-out simulated 1H NMR spectra, reporting overall and per-metabolite accuracies.

## When to use

Use when a trained NMRQNet CRNN model must be scored on held-out simulated nine-metabolite 1H NMR spectra for presence/absence identification accuracy.

## When NOT to use

- The task is quantification of relative concentrations or spectral reconstruction rather than presence/absence identification — the quantification pipeline and its RMSE/Pearson/residual metrics apply instead
- The spectra are real experimental plasma samples rather than the simulated nine-metabolite test split, since accuracy benchmarks reported here are defined on simulated data
- A single metabolite's signal must be inspected manually for false-positive calls — NMRQNet's false positives require a final visualization check rather than this aggregate accuracy metric

## Inputs

- Simulated nine-metabolite 1H NMR spectra test split from NMRQNet's data-augmentation simulation pipeline
- Trained CRNN (CNN + GRU) identification model from the NMRQNet optimization pipeline
- Ground-truth per-metabolite presence/absence labels for the simulated test samples

## Outputs

- Overall presence/absence identification accuracy on the simulated test set (expected ~92% of test samples fully correct)
- Per-metabolite average identification accuracies (expected range 0.978–0.995, Supplementary Tables 1–2)
- identification_accuracy.csv results table

## How to apply

Generate the simulated nine-metabolite 1H NMR training and test splits with NMRQNet's data-augmentation simulation pipeline, which introduces positional and pattern variations so the CRNN generalizes beyond fixed peak positions. Format the spectra as the NMRQNet csv input (chemical shift values in the first column, one spectrum per sample in each subsequent column) and train the CRNN (CNN + GRU) with the NMRQNet optimization pipeline. Apply the trained model to the held-out simulated test set to obtain per-metabolite presence/absence predictions. Compute the overall presence/absence accuracy plus per-metabolite average accuracies, and check them against the reported 92% overall accuracy and the 0.978–0.995 per-metabolite range (Supplementary Tables 1–2). Save the accuracy results table as identification_accuracy.csv.

## Related tools

- **NMRQNet** (Provides the data-augmentation simulation pipeline for generating nine-metabolite 1H NMR training/test splits, the csv input format, the CRNN (CNN + GRU) identification model and optimization pipeline, and the accuracy evaluation) — github.com/LiuzLab/NMRQNet

## Examples

```
jupyter notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- Overall presence/absence accuracy on the simulated test set reproduces the reported value of 92% of test samples correctly predicted for all nine metabolites
- Per-metabolite average accuracies fall within the reported 0.978–0.995 range (Supplementary Tables 1–2)
- Input csv has chemical shift values in the first column and one spectrum per sample in each subsequent column, matching NMRQNet's expected format
- Test spectra show positional and pattern variations relative to training data, confirming the augmentation pipeline was applied before the split
- Results table saved as identification_accuracy.csv with overall and per-metabolite accuracy entries

## Limitations

- NMRQNet regularizes less on false-positive predictions because the loss focuses on eliminating residuals between original and reconstructed spectra, so accurate presence/absence metrics should be paired with a final visualization check to filter false positives
- The 92% accuracy figure applies to the simulated nine-metabolite test set generated from the reference library, not to experimental plasma spectra
- Accuracy is limited to the nine-metabolite reference set rather than the broader 39-metabolite IVDr panel

## Evidence

- [results] Simulated test-set presence/absence accuracy benchmark: [evidence span withheld — anchor status: not_found]
