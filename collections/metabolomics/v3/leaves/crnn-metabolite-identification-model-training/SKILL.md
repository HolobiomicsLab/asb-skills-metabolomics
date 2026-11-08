---
name: crnn-metabolite-identification-model-training
description: Use when the goal is metabolite presence/absence identification from
  1H NMR spectra using a nine-metabolite reference set, and simulated augmented spectra
  (NMRQNet pipeline) are available for training and testing.
when_to_use_negative:
- The task is quantification (reconstructing spectra and estimating relative concentrations,
  scored by RMSE/Pearson correlation) rather than presence/absence identification.
- The input spectra are experimental plasma samples (e.g. EDTA/heparin/citrate plasma)
  rather than NMRQNet-simulated spectra; this skill's 92% accuracy applies to the
  simulated test set.
- The metabolite library differs substantially from the nine-metabolite reference
  set, so the reported 0.978–0.995 per-metabolite accuracies no longer apply.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0593
tools:
- name: NMRQNet
  role: Provides the data-augmentation simulation pipeline for generating nine-metabolite
    spectra, the csv input format, the CRNN (CNN + GRU) identification model and optimization
    pipeline, and the accuracy evaluation
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:40ccd01ffa82ba4445103f9a6e65af61758affe8463954a5621a424f3ab9471f
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# crnn-metabolite-identification-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains NMRQNet's CRNN (CNN + GRU) model on simulated nine-metabolite 1H NMR spectra to predict presence/absence of each metabolite, and evaluates per-metabolite and overall identification accuracy on a held-out simulated test set.

## When to use

Use when the goal is metabolite presence/absence identification from 1H NMR spectra using a nine-metabolite reference set, and simulated augmented spectra (NMRQNet pipeline) are available for training and testing.

## When NOT to use

- The task is quantification (reconstructing spectra and estimating relative concentrations, scored by RMSE/Pearson correlation) rather than presence/absence identification.
- The input spectra are experimental plasma samples (e.g. EDTA/heparin/citrate plasma) rather than NMRQNet-simulated spectra; this skill's 92% accuracy applies to the simulated test set.
- The metabolite library differs substantially from the nine-metabolite reference set, so the reported 0.978–0.995 per-metabolite accuracies no longer apply.

## Inputs

- Simulated nine-metabolite 1H NMR spectra training split (NMRQNet csv: chemical shifts in first column, one spectrum per subsequent column)
- Held-out simulated test split in the same NMRQNet csv format
- Nine-metabolite reference set defining the presence/absence labels

## Outputs

- Presence/absence predictions for each of the nine metabolites on the test set
- Overall and per-metabolite identification accuracy table saved as identification_accuracy.csv

## How to apply

Generate the simulated nine-metabolite 1H NMR spectra training and test splits with NMRQNet's data-augmentation simulation pipeline, which deliberately introduces positional and pattern variations so the model generalizes beyond fixed peak positions. Format spectra as the NMRQNet csv input: chemical shift values in the first column and one spectrum per sample in each subsequent column. Train the CRNN (CNN + GRU) identification model on the simulated spectra with the NMRQNet optimization pipeline, then apply it to the held-out simulated test set to obtain metabolite presence/absence predictions. Compute overall presence/absence accuracy and per-metabolite average accuracies, checking against the reported 92% overall accuracy and the 0.978–0.995 per-metabolite range (Supplementary Tables 1–2), and save the results table as identification_accuracy.csv. Because NMRQNet regularizes weakly on false-positive predictions, any deployment on real spectra additionally requires a final manual visualization check to filter false positives.

## Related tools

- **NMRQNet** (Provides the data-augmentation simulation pipeline for generating nine-metabolite spectra, the csv input format, the CRNN (CNN + GRU) identification model and optimization pipeline, and the accuracy evaluation) — github.com/LiuzLab/NMRQNet

## Examples

```
jupyter notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- Overall presence/absence accuracy on the held-out simulated test set is approximately 92% of test samples
- Per-metabolite average accuracies fall within the 0.978–0.995 range reported in Supplementary Tables 1–2
- Input csv matches the NMRQNet format: first column is chemical shift values, one column per spectrum
- Results table saved as identification_accuracy.csv contains both overall and per-metabolite accuracies
- Training spectra show positional and pattern variations introduced by the simulation pipeline (not fixed peak positions)

## Limitations

- NMRQNet regularizes less on false-positive predictions, so predicted presence calls require a final manual visualization check to filter false positives
- The reported accuracies are on simulated test data; performance on real plasma spectra is not established by this evaluation
- The data-augmentation simulation pipeline exists to overcome the data-shortage issue for deep learning in metabolomics; without augmentation the model likely fails to generalize beyond fixed peak positions

## Evidence

- [readme] The first column is chemical shift values: "The first column is chemical shift values"
- [readme] For the rest of columns, each column is the 1H NMR spectrum for one plasma sample: "For the rest of columns, each column is the 1H NMR spectrum for one plasma sample"
- [discussion] Overall, the data augmentation implemented simulation pipeline overcame the data shortage issue for deep learning in the metabolomics area.: "Overall, the data augmentation implemented simulation pipeline overcame the data shortage issue for deep learning in the metabolomics area."
