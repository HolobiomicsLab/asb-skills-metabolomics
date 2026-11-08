---
name: neural-spectrum-prediction-mode-configuration
description: Use when a neural mass-spectra prediction model must be configured with
  forward-only, reverse-only, or bidirectional output heads, or when comparing prediction
  modes by library-matching recall on EI spectra.
when_to_use_negative:
- The input is already observed library spectra requiring only library matching with
  no predicted spectra — no prediction-mode configuration is needed
- The target ionization method is not electron ionization and no retrained NEIMS variant
  is available (the approach must be retrained for other ionization methods)
- Isotope intensity peaks in ion fragments must be modeled accurately — NEIMS lacks
  a method to model isotope peaks, only a small shift is applied
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Tensorflow
  role: Constructs and trains the NEIMS MLP and its forward/reverse/bidirectional
    output heads
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates the additive ECFP fingerprints (length 4096, radius 2) that are
    the model inputs
- name: Adam optimizer
  role: Stochastic gradient descent optimizer for the MLP parameters during training
- name: Vizier
  role: Hyperparameter tuning used to select the optimal MLP architecture
- name: NEIMS
  role: The spectral prediction model whose prediction mode (forward/reverse/bidirectional)
    is being configured
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:7d1e7b8766bc35da70d96e4268e5073e06759217c1687c3ffb35006c6a57cdad
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# neural-spectrum-prediction-mode-configuration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Configure and compare NEIMS spectral-prediction modes (forward, reverse, bidirectional) on additive ECFP(4096, radius 2) inputs, then evaluate each mode via library-matching recall@10 with weighted cosine similarity on the NIST 2017 setup.

## When to use

Use when a neural mass-spectra prediction model must be configured with forward-only, reverse-only, or bidirectional output heads, or when comparing prediction modes by library-matching recall on EI spectra.

## When NOT to use

- The input is already observed library spectra requiring only library matching with no predicted spectra — no prediction-mode configuration is needed
- The target ionization method is not electron ionization and no retrained NEIMS variant is available (the approach must be retrained for other ionization methods)
- Isotope intensity peaks in ion fragments must be modeled accurately — NEIMS lacks a method to model isotope peaks, only a small shift is applied

## Inputs

- Molecular structures as SMILES/SDF (e.g. pentachlorobenzene.sdf) converted to additive ECFP fingerprints of length 4096, radius 2 via RDKit
- NIST 2017 replicates library (held-out half as query set)
- NIST main library spectra for constructing the augmented reference library
- Trained NEIMS model weights (forward affine head, reverse residual-index head, bidirectional sigmoid-gate variants)

## Outputs

- Predicted mass spectra vectors (intensity per integral m/z bin) per molecule per prediction mode
- Per-query rank of the correct candidate spectrum under each mode
- results_table.csv tabulating recall@10 per prediction mode

## How to apply

Build the NEIMS model in TensorFlow on top of additive ECFP fingerprints of length 4096 with radius 2 (generated with RDKit), using the MLP architecture of seven layers of 2000 nodes with residual connections, ReLU activation, and dropout 0.25, trained for 100000 steps at batch size 100 with Adam. Configure the prediction mode: forward prediction computes output at bin i (eq 3); reverse prediction indexes output relative to the molecular mass minus the neutral-loss offset (eq 4, with a small shift for isotopes); bidirectional prediction merges both via a coordinate-wise sigmoid gate (eq 5). Preprocess the held-out half of the NIST 2017 replicates library as the query set and build the augmented reference library by removing query-molecule spectra from the NIST main library and replacing them with each mode's predictions. Run library matching with weighted cosine similarity (eq 1) and the mass filter set to a tolerance of 5 Da of the query molecule's mass, record the correct candidate's rank per query per mode, and compute recall@10 (fraction of queries whose correct spectrum ranks at or below rank 10). Validate that bidirectional prediction yields the highest recall@10 (paper reports 91.8% for full bidirectional NEIMS), exceeding forward-only and reverse-only baselines, and save the comparison as results_table.csv.

## Related tools

- **Tensorflow** (Constructs and trains the NEIMS MLP and its forward/reverse/bidirectional output heads) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates the additive ECFP fingerprints (length 4096, radius 2) that are the model inputs)
- **Adam optimizer** (Stochastic gradient descent optimizer for the MLP parameters during training)
- **Vizier** (Hyperparameter tuning used to select the optimal MLP architecture)
- **NEIMS** (The spectral prediction model whose prediction mode (forward/reverse/bidirectional) is being configured) — github.com/brain-research/deep-molecular-massspec

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Bidirectional model achieves the highest recall@10 of the three modes; paper reports 91.8% for the full bidirectional NEIMS
- Bidirectional prediction improves recall@10 accuracy by 30% over forward-only and reverse-only modes for both the linear regression and MLP models
- Mass filter with 5 Da tolerance reduces the library to a median of 6696 spectra per query molecule
- Per-query correct-spectrum rank is recorded for every query under each mode before aggregating recall@10
- results_table.csv is produced with recall@10 rows per prediction mode including the linear-regression baseline

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments; only a small shift for isotopes is applied in reverse prediction
- Recall is bounded by library coverage: if the sample molecule is not in the (augmented) library, correct identification is impossible
- The evaluation replaces query-molecule spectra in the NIST main library with predictions, so results depend on the library-matching task construction and the 5 Da mass filter

## Evidence

- [results] Bidirectional prediction improves recall@10 accuracy by 30% for both the linear regression and MLP models: [evidence span withheld — anchor status: not_found]
- [readme] library construction: [evidence span withheld — anchor status: external_doc]
