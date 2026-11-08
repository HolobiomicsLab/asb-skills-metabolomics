---
name: scaffold-based-dataset-splitting
description: Use when evaluating MS/MS spectral-prediction or retrieval models on
  NIST20-like datasets and you need a split that prevents highly similar scaffold-mates
  from leaking into the test set, making generalization across chemical classes a
  fair test.
when_to_use_negative:
- When the goal is a permissive random-split benchmark and scaffold leakage is acceptable
  — the stricter scaffold split will understate model performance.
- When the dataset lacks molecular structures from which Murcko scaffolds can be derived,
  since the split cannot be computed.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ICEBERG
  role: Spectral-prediction model evaluated under the scaffold split via its Generate
    (DAG fragment generator, train_gen.py) and Score (intensity model, train_inten.py)
    modules with a fragment-generation threshold sweep, predictions via predict_smis.py
  repo: https://github.com/samgoldman97/ms-pred
- name: MassFormer
  role: Graph-transformer baseline trained and predicted under the same covariates
    and hyperparameter sweep settings (run_scripts/massformer_model 01_run_massformer_train.sh,
    02_predict_massformer.py) for the scaffold-split comparison
  repo: https://github.com/Roestlab/massformer
- name: SCARF
  role: Two-part subformula-autoregression baseline (train_gen.py then train_inten.py,
    predictions via predict_smis.py) evaluated on the scaffold split
  repo: https://github.com/samgoldman97/ms-pred
- name: spec_pred_eval.py
  role: Analysis script (analysis/spec_pred_eval.py) that scores predictions against
    reference spectra to compute average cosine similarity on the scaffold test split
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:bd0501ae08052bd90fc6064e9558cd75762593180acd95a19ae6e5a6b5d809b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - MassFormer
  - Python
  - SCARF
  - spec_pred_eval.py
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scaffold-based-dataset-splitting

## Summary

Split a mass spectrometry / machine-learning benchmark dataset so that test-set molecules belong to smaller Murcko scaffold clusters held out from training, exposing whether a spectral-prediction model generalizes across chemical scaffolds rather than memorizing scaffold-mates. Used in the ICEBERG work to compare ICEBERG, MassFormer, and SCARF under a stricter split than random.

## When to use

Use when evaluating MS/MS spectral-prediction or retrieval models on NIST20-like datasets and you need a split that prevents highly similar scaffold-mates from leaking into the test set, making generalization across chemical classes a fair test.

## When NOT to use

- When the goal is a permissive random-split benchmark and scaffold leakage is acceptable — the stricter scaffold split will understate model performance.
- When the dataset lacks molecular structures from which Murcko scaffolds can be derived, since the split cannot be computed.

## Inputs

- Processed spectral dataset (labels.tsv, mgf_files, spec_files.hdf5) e.g. ms-pred NIST20
- Molecule SMILES/InChI structures for Murcko scaffold extraction
- Model predictions on scaffold-split test set (ICEBERG, MassFormer, SCARF)

## Outputs

- Scaffold-based train/test split assignment per molecule
- Per-model average cosine similarity on the scaffold test split (e.g. scaffold_split_cosine_results.csv)
- Model-vs-model margin comparison under scaffold splitting

## How to apply

Load the processed dataset (e.g. ms-pred NIST20 with labels.tsv, mgf_files, and spec_files.hdf5) and assign each molecule to a Murcko scaffold cluster; select the scaffold split variant that places smaller scaffold clusters into the test set, which is the stricter setting. Then train each competing model (ICEBERG two-part pipeline via train_gen.py and train_inten.py with a fragment-generation threshold sweep, MassFormer via run_scripts/massformer_model 01_run_massformer_train.sh and 02_predict_massformer.py, SCARF via its train_gen.py/train_inten.py pipeline) under identical covariates and hyperparameter sweep settings. Predict spectra for the scaffold-split test set with each model's predict_smis.py, evaluate with analysis/spec_pred_eval.py to compute average cosine similarity, and aggregate per-model scores (e.g. scaffold_split_cosine_results.csv) to quantify margins. This checks whether the model's advantage persists when scaffold-mates are absent from training, as in the ICEBERG finding that it beats MassFormer by 0.017 and SCARF by 0.03 cosine similarity on this split.

## Related tools

- **ICEBERG** (Spectral-prediction model evaluated under the scaffold split via its Generate (DAG fragment generator, train_gen.py) and Score (intensity model, train_inten.py) modules with a fragment-generation threshold sweep, predictions via predict_smis.py) — https://github.com/samgoldman97/ms-pred
- **MassFormer** (Graph-transformer baseline trained and predicted under the same covariates and hyperparameter sweep settings (run_scripts/massformer_model 01_run_massformer_train.sh, 02_predict_massformer.py) for the scaffold-split comparison) — https://github.com/Roestlab/massformer
- **SCARF** (Two-part subformula-autoregression baseline (train_gen.py then train_inten.py, predictions via predict_smis.py) evaluated on the scaffold split) — https://github.com/samgoldman97/ms-pred
- **spec_pred_eval.py** (Analysis script (analysis/spec_pred_eval.py) that scores predictions against reference spectra to compute average cosine similarity on the scaffold test split) — https://github.com/samgoldman97/ms-pred

## Examples

```
python run_scripts/iceberg/nist23/01_run_iceberg_gen.py --split scaffold && python run_scripts/massformer_model/02_predict_massformer.py --split scaffold && python analysis/spec_pred_eval.py --out scaffold_split_cosine_results.csv
```

## Evaluation signals

- ICEBERG exceeds MassFormer by ~0.017 and SCARF by ~0.03 average cosine similarity on the scaffold split (2% and 4% improvements).
- All compared models use identical covariates and hyperparameter sweep settings, so differences reflect the split, not tuning.
- Per-model results aggregate into scaffold_split_cosine_results.csv with average cosine similarity computed by analysis/spec_pred_eval.py.
- The test set is composed of smaller Murcko scaffold clusters, confirming the stricter variant was selected.

## Limitations

- The scaffold split makes the evaluation stricter than a random split, so reported margins will differ from random-split results and may understate raw model performance.
- DAG fragment generation cost (~1 CPU second per molecule, ~10x SCARF) limits throughput of the ICEBERG pipeline being evaluated.
- Model-generated fragment explanations are not guaranteed to be the correct physical explanations due to equivalent-mass atom and bond rearrangements and cosine-similarity training.

## Evidence

- [results] Strict scaffold split margins over MassFormer and SCARF: [evidence span withheld — anchor status: not_found]
- [methods] Dataset files for scaffold-split evaluation: [evidence span withheld — anchor status: not_found]
- [results] Scaffold split assigns smaller scaffold clusters to test: [evidence span withheld — anchor status: not_found]
- [methods] Evaluation metric for the split: [evidence span withheld — anchor status: not_found]
