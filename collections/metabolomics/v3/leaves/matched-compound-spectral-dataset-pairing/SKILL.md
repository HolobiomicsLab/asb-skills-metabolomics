---
name: matched-compound-spectral-dataset-pairing
description: Use when you must compare model performance across instrument types (Q-TOF
  vs Orbitrap) on identical compounds, requiring matched spectral pairs for the same
  molecules rather than disjoint dataset splits.
when_to_use_negative:
- You need maximum generalization performance rather than a controlled instrument-effect
  comparison — train on the full mixed Q-TOF and Orbitrap corpus instead of matched
  pairs
- 'Spectra fall outside the paper''s scope: fewer than five peaks, m/z above 1500,
  molecules with rare atoms, total atom count above 300, or uncommon precursor/adduct
  types'
- The goal is structure elucidation or adduct-type prediction — this skill only supports
  formula-level evaluation via missed-atom counts
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIDDLE
  role: Deep learning framework whose TCN models are trained on matched Q-TOF/Orbitrap
    pairs via train_tcn_gpus.py and evaluated for missed-atom counts
  repo: https://github.com/JosieHong/FIDDLE
- name: MoNA
  role: Source of Q-TOF spectra for one side of the matched compound pairs
- name: NIST23
  role: Source of Orbitrap spectra (downsampled) for the other side of the matched
    compound pairs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:f5e858f4411ec946927491d8c5718c1105e7d3a7465473b7f1e6d520a8fe5c3f
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
  - MoNA
  - NIST23
  - Q-TOF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# matched-compound-spectral-dataset-pairing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build instrument-matched MS/MS training pairs by linking MoNA Q-TOF spectra with downsampled NIST23 Orbitrap spectra for the same compounds, enabling controlled cross-instrument generalization tests for FIDDLE formula-prediction models. This isolates the effect of instrument type (Q-TOF vs Orbitrap) from compound identity.

## When to use

Use when you must compare model performance across instrument types (Q-TOF vs Orbitrap) on identical compounds, requiring matched spectral pairs for the same molecules rather than disjoint dataset splits.

## When NOT to use

- You need maximum generalization performance rather than a controlled instrument-effect comparison — train on the full mixed Q-TOF and Orbitrap corpus instead of matched pairs
- Spectra fall outside the paper's scope: fewer than five peaks, m/z above 1500, molecules with rare atoms, total atom count above 300, or uncommon precursor/adduct types
- The goal is structure elucidation or adduct-type prediction — this skill only supports formula-level evaluation via missed-atom counts

## Inputs

- MoNA Q-TOF tandem mass spectra (peak lists with m/z and intensity)
- NIST23 Orbitrap tandem mass spectra (downsampled)
- Compound identifier mappings linking Q-TOF and Orbitrap entries for the same molecules
- FIDDLE TCN config files (e.g., config/fiddle_tcn_orbitrap.yml and Q-TOF counterpart)

## Outputs

- Matched Q-TOF/Orbitrap training, validation, and test spectral pairs for identical compound sets
- Two trained FIDDLE TCN checkpoints (Orbitrap-trained and Q-TOF-trained models)
- 2x2 missed-atom matrix (same-instrument vs cross-instrument evaluation, e.g., fig4j_missed_atom_matrix.png)

## How to apply

Link MoNA Q-TOF spectra and downsampled NIST23 Orbitrap spectra for the same compounds, then apply the paper's filtering: exclude spectra with fewer than five peaks, confine m/z to (0, 1500], retain only molecules with frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), cap total atom count at 300, and keep only common precursor types (e.g., [M + H]+, [M + Na]+, [2M + H]+). Train an Orbitrap TCN with train_tcn_gpus.py and config/fiddle_tcn_orbitrap.yml without Gaussian-noise augmentation (noise addition lowered similarity for Orbitrap spectra), and a Q-TOF TCN with its config where augmentation (Gaussian noise, mean 0, sd 0.1) is optional. Evaluate each checkpoint on matched test spectra of BOTH instrument types, recording missed-atom counts per predicted formula, then assemble a 2x2 same-instrument vs cross-instrument missed-atom matrix. Rationale: matched compound sets ensure performance differences reflect spectral resolution and instrument characteristics, not molecular composition.

## Related tools

- **FIDDLE** (Deep learning framework whose TCN models are trained on matched Q-TOF/Orbitrap pairs via train_tcn_gpus.py and evaluated for missed-atom counts) — https://github.com/JosieHong/FIDDLE
- **MoNA** (Source of Q-TOF spectra for one side of the matched compound pairs)
- **NIST23** (Source of Orbitrap spectra (downsampled) for the other side of the matched compound pairs)

## Examples

```
python train_tcn_gpus.py --config config/fiddle_tcn_orbitrap.yml
```

## Evaluation signals

- The 2x2 missed-atom matrix reproduces the Fig. 4j pattern: the Orbitrap-trained model incurs fewer missed atoms than the Q-TOF-trained model when tested on either instrument type
- Matched train/test splits contain no shared compounds (splits by molecular canonical SMILES)
- Filtered compounds satisfy all constraints: >=5 peaks, m/z in (0, 1500], frequent atoms only, total atom count <= 300, common precursor types
- Orbitrap training config runs with data augmentation disabled, since Gaussian noise addition is not applied for Orbitrap training
- Missed-atom counts are recorded per predicted formula on BOTH instrument types' matched test spectra

## Limitations

- Experimental condition effects are not completely eliminated; accuracy varies across conditions, especially those with less training data
- No data augmentation is applied for Orbitrap training because noise addition led to lower spectral similarity for Orbitrap spectra
- Orbitrap spectra are downsampled to enable pairing, so cross-instrument conclusions depend on the downsampled representation of higher-resolution data

## Evidence
