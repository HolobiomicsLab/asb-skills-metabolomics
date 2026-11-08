---
name: deep-learning-test-set-accuracy-benchmarking
description: Use when a trained NMRQNet CRNN model must be evaluated on a held-out
  simulated nine-metabolite test set, and per-metabolite accuracy values must be compared
  against reported benchmarks (92% overall, 0.978-0.995 per metabolite).
when_to_use_negative:
- The task is real-sample quantification on experimental plasma spectra (EDTA/heparin/citrate),
  not simulated-test-set identification benchmarking — NMRQNet false positives additionally
  require a final manual visualization check on real data.
- The goal is spectral reconstruction quality (RMSE, Pearson correlation, residuals)
  or biomarker discovery from estimated relative concentrations rather than presence/absence
  classification accuracy.
- The metabolite panel differs from the nine-metabolite reference set the simulated
  spectra were generated from, so the per-metabolite benchmark range 0.978-0.995 is
  not applicable.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: NMRQNet
  role: Provides the data-augmentation simulation pipeline that generates the nine-metabolite
    train/test spectra, the CRNN (CNN + GRU) identification model, the optimization
    pipeline used to train it, and the csv input format used for evaluation
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:0336cfd998b230081c5d48eea649cd59623f9f6e2421b15b2a62a613ab3412ab
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# deep-learning-test-set-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a trained deep learning model (here NMRQNet's CRNN, i.e. CNN + GRU) by computing overall and per-class presence/absence identification accuracy on a held-out simulated 1H NMR test set, and saves the results as a CSV table for comparison against the paper's reported metrics.

## When to use

Use when a trained NMRQNet CRNN model must be evaluated on a held-out simulated nine-metabolite test set, and per-metabolite accuracy values must be compared against reported benchmarks (92% overall, 0.978-0.995 per metabolite).

## When NOT to use

- The task is real-sample quantification on experimental plasma spectra (EDTA/heparin/citrate), not simulated-test-set identification benchmarking — NMRQNet false positives additionally require a final manual visualization check on real data.
- The goal is spectral reconstruction quality (RMSE, Pearson correlation, residuals) or biomarker discovery from estimated relative concentrations rather than presence/absence classification accuracy.
- The metabolite panel differs from the nine-metabolite reference set the simulated spectra were generated from, so the per-metabolite benchmark range 0.978-0.995 is not applicable.

## Inputs

- Held-out simulated nine-metabolite 1H NMR test spectra in NMRQNet csv format (first column: chemical shift values; one column per sample spectrum)
- Trained CRNN (CNN + GRU) identification model from the NMRQNet optimization pipeline
- Ground-truth presence/absence labels for the nine metabolites per test sample

## Outputs

- Overall presence/absence identification accuracy on the simulated test set (expected ~92%)
- Per-metabolite average accuracy values for all nine metabolites (expected 0.978-0.995)
- identification_accuracy.csv results table

## How to apply

First generate the simulated nine-metabolite 1H NMR spectra train/test splits using NMRQNet's data-augmentation simulation pipeline, which deliberately introduces positional and pattern variations so the model generalizes beyond fixed peak positions. Format the spectra as the NMRQNet csv input: chemical shift values in the first column, one spectrum per sample in each subsequent column (see NMRQNet/data/tutorial_data/control_human_plasma.csv as the format reference). After building and training the CRNN (CNN + GRU) identification model via the NMRQNet optimization pipeline, apply it to the held-out simulated test set to obtain metabolite presence/absence predictions. Compute the overall presence/absence accuracy and the per-metabolite average accuracies, then check these against the reported benchmarks of 92% overall accuracy and 0.978-0.995 per-metabolite accuracy (Supplementary Tables 1-2). Save the final accuracy table as identification_accuracy.csv so the benchmark is reproducible and diffable.

## Related tools

- **NMRQNet** (Provides the data-augmentation simulation pipeline that generates the nine-metabolite train/test spectra, the CRNN (CNN + GRU) identification model, the optimization pipeline used to train it, and the csv input format used for evaluation) — github.com/LiuzLab/NMRQNet

## Examples

```
jupyter notebook NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb
```

## Evaluation signals

- Overall presence/absence accuracy on the held-out simulated test set reproduces the reported ~92% (all nine metabolites correctly predicted for 92% of test samples).
- Every per-metabolite average accuracy falls within the reported 0.978-0.995 range (Supplementary Tables 1-2).
- The saved output file is named identification_accuracy.csv and contains both overall and per-metabolite accuracy columns.
- Input spectra parse correctly as the NMRQNet csv format with chemical shift values in the first column and one spectrum per subsequent column.
- Train/test splits come from the data-augmentation simulation pipeline, so test spectra exhibit positional and pattern variations relative to fixed peak positions rather than being trivially identical to training spectra.

## Limitations

- The 92% overall and 0.978-0.995 per-metabolite accuracies apply to the simulated test set generated by the augmentation pipeline, not to real plasma samples.
- On real data, NMRQNet regularizes less on false-positive predictions, so a final manual visualization check is required to filter metabolites with false-positive predictions — a limitation that accuracy-on-simulated-data benchmarking does not expose.
- Results depend on the augmentation pipeline introducing sufficient positional and pattern variation; without it, the benchmark overstates generalization to real spectra.

## Evidence

- [supplementary] On the simulated test set, the CRNN model accurately predicted the presence and absence of all nine metabolites for 92% of the test samples.: [evidence span withheld — anchor status: not_found]
