---
name: cross-instrument-generalization-evaluation
description: Use when testing whether a formula-prediction model trained on one instrument's
  MS/MS spectra (e.g. Orbitrap) generalizes to another (e.g. Q-TOF) on identical compound
  sets.
when_to_use_negative:
- The analysis goal is top-1/top-5 accuracy benchmarking against SIRIUS or BUDDY rather
  than instrument-transfer behavior
- The compound sets differ between the two instruments (matched same-compound pairs
  are required for a fair 2x2 comparison)
- The spectra come from other instrument families (e.g. ion trap) not covered by the
  Q-TOF/Orbitrap training data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: FIDDLE
  role: Deep learning framework whose TCN models are trained per instrument type and
    evaluated cross-instrument via train_tcn_gpus.py
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Source instrument for MoNA spectra; one arm of the cross-instrument train/test
    matrix
- name: Orbitrap
  role: Source instrument for downsampled NIST23 spectra; the higher-resolution training
    arm of the comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:95cc28a7ffd4c62e2852f4e930097bcd7f6189e4eaae75ad54346a6debe854b5
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-instrument-generalization-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces FIDDLE's cross-instrument validation by training separate TCN models on matched Q-TOF (MoNA) and Orbitrap (NIST23, downsampled) spectra of the same compounds, then evaluating each checkpoint on both instrument types to assemble the 2x2 missed-atom matrix of Fig. 4j.

## When to use

Use when testing whether a formula-prediction model trained on one instrument's MS/MS spectra (e.g. Orbitrap) generalizes to another (e.g. Q-TOF) on identical compound sets.

## When NOT to use

- The analysis goal is top-1/top-5 accuracy benchmarking against SIRIUS or BUDDY rather than instrument-transfer behavior
- The compound sets differ between the two instruments (matched same-compound pairs are required for a fair 2x2 comparison)
- The spectra come from other instrument families (e.g. ion trap) not covered by the Q-TOF/Orbitrap training data

## Inputs

- MoNA Q-TOF MS/MS spectra for compounds with NIST23 counterparts
- Downsampled NIST23 Orbitrap MS/MS spectra for the same compounds
- config/fiddle_tcn_orbitrap.yml and corresponding Q-TOF TCN config
- Preprocessed spectrum tables filtered per paper criteria (>=5 peaks, m/z <= 1500, frequent atoms, <=300 atoms)

## Outputs

- Trained Orbitrap TCN checkpoint and Q-TOF TCN checkpoint
- Per-formula missed-atom counts for each (model, instrument) evaluation cell
- 2x2 missed-atom matrix figure (fig4j_missed_atom_matrix.png)

## How to apply

Build matched training pairs by linking MoNA Q-TOF spectra and downsampled NIST23 Orbitrap spectra of the same compounds, applying the paper's filters: at least five peaks per spectrum, m/z in (0, 1500], only frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), and a cap of 300 atoms per molecule. Train the Orbitrap TCN model with train_tcn_gpus.py using config/fiddle_tcn_orbitrap.yml without data augmentation, since Gaussian noise addition (mean 0, sd 0.1 on intensities) is not applied for Orbitrap; train the Q-TOF TCN with the corresponding Q-TOF config, where noise augmentation is optional. Evaluate each checkpoint on matched test spectra of BOTH instrument types, recording missed-atom counts per predicted formula. Assemble the 2x2 missed-atom matrix contrasting same-instrument vs cross-instrument evaluation and plot it as fig4j_missed_atom_matrix.png; the acceptance check is qualitative agreement of the matrix pattern with the published Fig. 4j.

## Related tools

- **FIDDLE** (Deep learning framework whose TCN models are trained per instrument type and evaluated cross-instrument via train_tcn_gpus.py) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Source instrument for MoNA spectra; one arm of the cross-instrument train/test matrix)
- **Orbitrap** (Source instrument for downsampled NIST23 spectra; the higher-resolution training arm of the comparison)

## Examples

```
python train_tcn_gpus.py --config config/fiddle_tcn_orbitrap.yml
```

## Evaluation signals

- The 2x2 matrix pattern matches Fig. 4j: the Orbitrap-trained model incurs fewer missed atoms than the Q-TOF-trained model on both Q-TOF and Orbitrap test data
- Both models are evaluated on identical matched compound test sets for both instruments
- The Orbitrap training run used no Gaussian-noise augmentation (mean 0, sd 0.1) while the Q-TOF run optionally used it
- All paired spectra satisfy the filters: >=5 peaks, m/z in (0, 1500], frequent atoms only, <=300 atoms per molecule

## Limitations

- Experimental-condition effects are not fully eliminated; FIDDLE accuracy varies across conditions, especially those with less training data
- No data augmentation was applied for Orbitrap training because noise addition led to lower similarity for Orbitrap spectra, so the two arms are not trained under identical augmentation settings
- Performance degrades on chimeric (co-eluting) spectra as spectral mixing increases

## Evidence

- [results] Orbitrap-trained model outperforms on both instruments: [evidence span withheld — anchor status: not_found]
