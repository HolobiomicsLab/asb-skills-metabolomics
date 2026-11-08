---
name: multi-model-evaluation-comparison
description: Use when several trained spectrum-prediction checkpoints (fragment-level
  and binned-vocabulary models) must be scored on identical held-out datasets to produce
  a fair, per-model accuracy table.
when_to_use_negative:
- Evaluating retrieval/structural elucidation rank accuracy (top-k candidate ranking)
  rather than spectrum-prediction cosine similarity — that task has a separate workflow
  (`src/ms_pred/retrieval/bootstrap_metrics.py`).
- Comparing models trained on different datasets or without matched hyperparameter
  sweep budgets, since the repo's fairness guarantee depends on equivalent settings
  across models.
- Working with the retired NPLIB1 dataset on the current ms-pred main branch — the
  legacy `iceberg_1.0` branch is required instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ICEBERG
  role: Fragment-level spectrum prediction model being benchmarked and reference implementation
  repo: https://github.com/samgoldman97/ms-pred
- name: MassFormer
  role: Graph-transformer baseline retrained and evaluated under equivalent settings
  repo: https://github.com/Roestlab/massformer
- name: SCARF
  role: Subformula-classification baseline predicting at the chemical-formula level
  repo: https://github.com/samgoldman97/ms-pred
- name: NEIMS
  role: FFN/GNN binned-spectrum baseline implemented in ms-pred
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Combinatorial baseline run via provided instructions without retraining
  repo: https://github.com/samgoldman97/ms-pred
- name: MetFrag
  role: Combinatorial fragmentation tool referenced among baselines
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: uv
  role: Environment manager used to create the ms-pred evaluation environment
  repo: https://docs.astral.sh/uv/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:14c61a2859f50b587b1ebad4321a582a401294605c8bae15a09ece0115b666f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MassFormer
  - MetFrag
  - NEIMS
  - Python
  - SCARF
  - uv
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# multi-model-evaluation-comparison

## Summary

Benchmark multiple MS/MS spectrum-prediction models (ICEBERG vs. MassFormer, FixedVocab, SCARF, NEIMS, CFM-ID, etc.) on the same test splits (NPLIB1, NIST20) by generating predicted spectra from each model and computing vectorized cosine similarity against true 0.1-Da binned spectra.

## When to use

Use when several trained spectrum-prediction checkpoints (fragment-level and binned-vocabulary models) must be scored on identical held-out datasets to produce a fair, per-model accuracy table.

## When NOT to use

- Evaluating retrieval/structural elucidation rank accuracy (top-k candidate ranking) rather than spectrum-prediction cosine similarity — that task has a separate workflow (`src/ms_pred/retrieval/bootstrap_metrics.py`).
- Comparing models trained on different datasets or without matched hyperparameter sweep budgets, since the repo's fairness guarantee depends on equivalent settings across models.
- Working with the retired NPLIB1 dataset on the current ms-pred main branch — the legacy `iceberg_1.0` branch is required instead.

## Inputs

- Trained model checkpoints for ICEBERG, SCARF, MassFormer, NEIMS (FFN/GNN), FixedVocab, and CFM-ID
- Processed NPLIB1 and NIST20 spectrum datasets with ground-truth spectra and test splits
- SMILES lists for each test-set molecule

## Outputs

- Predicted binned spectra files per model per dataset
- Per-model average cosine similarity values on NPLIB1 and NIST20 test splits
- Benchmark comparison table (CSV)

## How to apply

Set up the ms-pred environment via `uv sync --extra cu124`, load pretrained ICEBERG weights plus the NPLIB1 and NIST20 processed datasets, and run each model's predict script (e.g., `src/ms_pred/iceberg/predict_smis.py`, `run_scripts/scarf_model/05_predict_form_inten.py`, `run_scripts/massformer_model/02_predict_massformer.py`, `run_scripts/ffn_model/02_predict_ffn.py`, `run_scripts/gnn_model/02_predict_gnn.py`, `run_scripts/molnetms/02_predict_ffn.py`, and CFM-ID) to generate binned spectra predictions on the same test split. Evaluate all predictions against ground truth with the vectorized cosine similarity on 0.1-Da binned vectors implemented in `analysis/spec_pred_eval.py`. Average cosine similarity across each test split per model per dataset and assemble a per-model/per-dataset benchmark table as CSV. Apply identical covariates and hyperparameter sweep budgets across models, as the repo implements baselines 'using equivalent settings across models (i.e., same covariates, hyperparameter sweeps for each, etc.)', so differences reflect modeling rather than configuration.

## Related tools

- **ICEBERG** (Fragment-level spectrum prediction model being benchmarked and reference implementation) — https://github.com/samgoldman97/ms-pred
- **MassFormer** (Graph-transformer baseline retrained and evaluated under equivalent settings) — https://github.com/Roestlab/massformer
- **SCARF** (Subformula-classification baseline predicting at the chemical-formula level) — https://github.com/samgoldman97/ms-pred
- **NEIMS** (FFN/GNN binned-spectrum baseline implemented in ms-pred) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Combinatorial baseline run via provided instructions without retraining) — https://github.com/samgoldman97/ms-pred
- **MetFrag** (Combinatorial fragmentation tool referenced among baselines) — https://github.com/ipb-halle/MetFragRelaunched
- **uv** (Environment manager used to create the ms-pred evaluation environment) — https://docs.astral.sh/uv/

## Examples

```
python run_scripts/scarf_model/05_predict_form_inten.py && python analysis/spec_pred_eval.py --pred spec_preds/ --true data/spec_datasets/nist20/test.json
```

## Evaluation signals

- ICEBERG average cosine similarity on NPLIB1 ≈ 0.627 and on NIST20 ≈ 0.727
- MassFormer and FixedVocab each land near 0.568 on NPLIB1 (ICEBERG ~10% higher)
- SCARF lands near 0.726 on NIST20, essentially tied with ICEBERG (0.727)
- All models evaluated on the identical test split with 0.1-Da binned vectors via `analysis/spec_pred_eval.py`
- Each model row falls within reported ranges, confirming equivalent settings were applied

## Limitations

- ICEBERG inference costs roughly 1 CPU second per molecule — nearly an order of magnitude more than SCARF — so benchmarking many models at scale is time-consuming.
- Fair comparison requires retraining each baseline with the same covariates and hyperparameter sweeps; off-the-shelf checkpoints trained on different data are not directly comparable.
- NPLIB1 support has been retired on the ms-pred main branch; the legacy `iceberg_1.0` branch must be used for those evaluations.
- NIST20/23 is a commercial dataset requiring purchase and separate export/processing via `reformat_nist_lcmsms_sdf.py`.

## Evidence

- [results] ICEBERG achieves an average cosine similarity of 0.627 on NPLIB1, outperforming MassFormer and FixedVocab: [evidence span withheld — anchor status: not_found]
- [methods] vectorized cosine similarity with 0.1-Da binned vectors, implemented in the analysis module (analysis/spec_pred_eval.py): [evidence span withheld — anchor status: not_found]
