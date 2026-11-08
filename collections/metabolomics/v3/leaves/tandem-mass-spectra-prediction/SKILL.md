---
name: tandem-mass-spectra-prediction
description: Use when you have a molecular structure dataset (e.g., NIST20 labels.tsv
  + mgf_files + spec_files.hdf5) and need to predict MS/MS spectra or compare spectrum
  simulators under matched covariates and hyperparameter sweeps.
when_to_use_negative:
- You only need a FAST heuristic for high-throughput screening — ICEBERG costs ~1
  CPU second per molecule, nearly an order of magnitude more than SCARF.
- You need guaranteed-correct physical fragment explanations — the paper notes there
  are no guarantees that generated fragment explanations are the correct physical
  explanations due to equivalent-mass atom and bond rearrangements.
- You are working with the NPLIB1/CANOPUS dataset on the main branch — support for
  NPLIB1 is retired; use the legacy iceberg_1.0 branch instead.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ICEBERG
  role: 'Two-part hybrid model: DAG fragment generator (train_gen.py) + Transformer
    intensity model (train_inten.py) that predicts MS/MS spectra at the fragment level'
  repo: https://github.com/samgoldman97/ms-pred
- name: MassFormer
  role: Graph-transformer baseline for spectrum prediction, trained/predicted via
    run_scripts/massformer_model under matched covariates and hyperparameter sweeps
  repo: https://github.com/Roestlab/massformer
- name: SCARF
  role: Subformula autoregressive baseline (prefix-tree generator train_gen.py + train_inten.py)
    predicting spectra at the chemical formula level
- name: Python
  role: Runs the ms-pred pipeline scripts (training, prediction, threshold sweeping,
    and spec_pred_eval.py cosine-similarity evaluation)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:84a539bf9c28c9ab3ad4e9fac655f3005083b4d177ac947c7a80d459bbe1bfda
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - MassFormer
  - Python
  - SCARF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# tandem-mass-spectra-prediction

## Summary

Predict molecular tandem MS/MS spectra from structures using the ICEBERG two-part pipeline (a DAG fragment generator followed by a Transformer intensity model), alongside baselines like MassFormer and SCARF, and evaluate predictions by average cosine similarity against reference spectra.

## When to use

Use when you have a molecular structure dataset (e.g., NIST20 labels.tsv + mgf_files + spec_files.hdf5) and need to predict MS/MS spectra or compare spectrum simulators under matched covariates and hyperparameter sweeps.

## When NOT to use

- You only need a FAST heuristic for high-throughput screening — ICEBERG costs ~1 CPU second per molecule, nearly an order of magnitude more than SCARF.
- You need guaranteed-correct physical fragment explanations — the paper notes there are no guarantees that generated fragment explanations are the correct physical explanations due to equivalent-mass atom and bond rearrangements.
- You are working with the NPLIB1/CANOPUS dataset on the main branch — support for NPLIB1 is retired; use the legacy iceberg_1.0 branch instead.

## Inputs

- Processed NIST20 dataset (labels.tsv, mgf_files, spec_files.hdf5 under ms-pred/data/spec_datasets/nist20)
- Molecular structures (SMILES/SDF) for prediction, e.g., exported NIST .SDF processed via reformat_nist_lcmsms_sdf.py
- Subformulae assignments from data_scripts/forms/assign_subformulae.py
- Reference experimental MS/MS spectra for evaluation

## Outputs

- Predicted tandem mass spectra per input molecule (from predict_smis.py)
- Per-model average cosine similarity scores on the test split (e.g., scaffold_split_cosine_results.csv)
- Trained ICEBERG generator and intensity model checkpoints (plus swept generation threshold)

## How to apply

Set up the ms-pred environment with `uv sync --extra cu124` (or cu118/cpu) and preprocess data with assign_subformulae.py before training. Run the ICEBERG pipeline in two parts: first train the DAG fragment generator (src/ms_pred/iceberg/train_gen.py), sweep the fragment-generation threshold (e.g., 02_sweep_gen_thresh.py) to pick the number of fragments to generate, then train the intensity model (train_inten.py), which is trained to maximize a vectorized cosine similarity; predict spectra with predict_smis.py. Train baselines (MassFormer graph transformer, SCARF prefix-tree + intensity model) with equivalent settings — same covariates and hyperparameter sweeps — for fair comparison. Evaluate all predictions against reference spectra with analysis/spec_pred_eval.py to compute average cosine similarity, then aggregate per-model results (e.g., into a scaffold_split_cosine_results.csv). For a robustness check, re-evaluate under a Murcko scaffold split (smaller scaffold clusters assigned to the test set) rather than a random split.

## Related tools

- **ICEBERG** (Two-part hybrid model: DAG fragment generator (train_gen.py) + Transformer intensity model (train_inten.py) that predicts MS/MS spectra at the fragment level) — https://github.com/samgoldman97/ms-pred
- **MassFormer** (Graph-transformer baseline for spectrum prediction, trained/predicted via run_scripts/massformer_model under matched covariates and hyperparameter sweeps) — https://github.com/Roestlab/massformer
- **SCARF** (Subformula autoregressive baseline (prefix-tree generator train_gen.py + train_inten.py) predicting spectra at the chemical formula level)
- **Python** (Runs the ms-pred pipeline scripts (training, prediction, threshold sweeping, and spec_pred_eval.py cosine-similarity evaluation))

## Examples

```
python data_scripts/forms/assign_subformulae.py
```

## Evaluation signals

- Average cosine similarity on the scaffold test split shows ICEBERG exceeding MassFormer by ~0.017 and SCARF by ~0.03 (2% and 4% improvements).
- All models were trained with equivalent settings (same covariates, hyperparameter sweeps for each) — verify comparison fairness before interpreting deltas.
- Fragment-generation threshold sweep (02_sweep_gen_thresh.py) is completed and the swept value is fixed before training the intensity model.
- spec_pred_eval.py runs cleanly over every model's predictions against the reference spectra without missing entries.
- Subformulae files were preprocessed by assign_subformulae.py before training (training should fail or produce invalid inputs otherwise).

## Limitations

- No guarantee that generated fragment explanations are the correct physical explanations, due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective.
- Higher computational cost: roughly 1 CPU second per molecule, nearly an order of magnitude more than other neural approaches like SCARF.
- NIST20/23 is a commercial dataset requiring purchase; pretrained ICEBERG/GLACIER weights for NIST are only shared upon proof of license.
- NPLIB1 (CANOPUS) dataset support is retired from the main branch — legacy code lives on the iceberg_1.0 branch.

## Evidence
