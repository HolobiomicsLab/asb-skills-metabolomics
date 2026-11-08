---
name: missed-atom-error-matrix-analysis
description: Use when comparing whether a formula-identification model trained on
  one instrument type (Q-TOF vs Orbitrap) generalizes to the other, using identical
  compound sets and per-formula missed-atom counts as the error metric.
when_to_use_negative:
- When the goal is formula identification accuracy (top-1/top-5) rather than instrument-generalization
  error analysis
- When no matched compound sets exist across Q-TOF and Orbitrap sources, so cross-instrument
  comparisons would be confounded by compound differences
- When spectra fall outside the paper's data space, e.g. fewer than five peaks, m/z
  above 1500, or molecules with atom counts above 300 or non-frequent atoms
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning framework whose Q-TOF and Orbitrap TCN models are trained and
    evaluated to compute missed-atom errors across instruments
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type providing one arm of the matched training/test spectra and
    one trained model
- name: Orbitrap
  role: High-resolution instrument type providing the second arm of matched spectra;
    its model learns from higher-resolution spectra
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:421738f969d857b210ce487c686722c69bbfc859e2629e4904afc86ea374c203
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - Orbitrap
  - Q-TOF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Missed-atom error matrix analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies cross-instrument generalization of FIDDLE formula predictors by evaluating a Q-TOF-trained and an Orbitrap-trained TCN model on matched spectra of both instrument types, assembling a 2x2 matrix of missed atoms per predicted formula (reproducing Fig. 4j).

## When to use

Use when comparing whether a formula-identification model trained on one instrument type (Q-TOF vs Orbitrap) generalizes to the other, using identical compound sets and per-formula missed-atom counts as the error metric.

## When NOT to use

- When the goal is formula identification accuracy (top-1/top-5) rather than instrument-generalization error analysis
- When no matched compound sets exist across Q-TOF and Orbitrap sources, so cross-instrument comparisons would be confounded by compound differences
- When spectra fall outside the paper's data space, e.g. fewer than five peaks, m/z above 1500, or molecules with atom counts above 300 or non-frequent atoms

## Inputs

- MoNA Q-TOF MS/MS spectra with assigned molecular formulas
- NIST23 Orbitrap MS/MS spectra (downsampled) for the same compounds
- Trained FIDDLE TCN checkpoints (Q-TOF-trained and Orbitrap-trained)
- Config files (e.g. config/fiddle_tcn_orbitrap.yml and Q-TOF equivalent)

## Outputs

- Per-formula missed-atom counts for each model on each instrument's test spectra
- 2x2 missed-atom matrix contrasting same-instrument vs cross-instrument evaluation
- fig4j_missed_atom_matrix.png reproducing Fig. 4j

## How to apply

Build matched training/testing pairs by linking MoNA Q-TOF spectra with downsampled NIST23 Orbitrap spectra for the same compounds, applying the paper's filters (at least five peaks, m/z in (0, 1500], frequent atoms only, atom count capped at 300). Train the Orbitrap TCN with train_tcn_gpus.py and config/fiddle_tcn_orbitrap.yml without data augmentation, since noise addition lowered similarity for Orbitrap spectra; train the Q-TOF TCN with the Q-TOF config where Gaussian noise (mean 0, sd 0.1) augmentation on intensities is optional. Evaluate each checkpoint on matched test spectra of BOTH instrument types, recording missed-atom counts per predicted formula, respecting the 10 ppm (Q-TOF) and 5 ppm (Orbitrap) precursor mass tolerances. Assemble the 2x2 missed-atom matrix (same-instrument vs cross-instrument cells) and plot it as fig4j_missed_atom_matrix.png for comparison with the published figure.

## Related tools

- **FIDDLE** (Deep learning framework whose Q-TOF and Orbitrap TCN models are trained and evaluated to compute missed-atom errors across instruments) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type providing one arm of the matched training/test spectra and one trained model)
- **Orbitrap** (High-resolution instrument type providing the second arm of matched spectra; its model learns from higher-resolution spectra)

## Examples

```
python train_tcn_gpus.py --config config/fiddle_tcn_orbitrap.yml
```

## Evaluation signals

- The Orbitrap-trained model incurs fewer missed atoms than the Q-TOF-trained model when tested on either instrument type, matching the Fig. 4j pattern
- The 2x2 matrix cells each derive from the same matched test compound sets, so cross-instrument cells are directly comparable
- Preprocessed spectra satisfy the filters: >=5 peaks, m/z in (0, 1500], frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) only, <=300 atoms per molecule
- Predicted formulas align with precursor mass within 10 ppm (Q-TOF) and 5 ppm (Orbitrap) tolerance
- No Gaussian-noise augmentation is present in the Orbitrap training run, per the paper's finding that it lowered similarity

## Limitations

- Experimental condition effects are not completely eliminated; accuracy varies across conditions, especially those with less training data
- Noise augmentation is unsuitable for Orbitrap spectra: 'the training size for Orbitrap is sufficiently large, we did not apply data augmentation'
- Acceptance is qualitative (agreement of the matrix pattern with the published figure), not a strict numeric threshold

## Evidence
