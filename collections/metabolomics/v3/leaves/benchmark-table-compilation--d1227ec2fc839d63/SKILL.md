---
name: benchmark-table-compilation
description: Use when you must compare a newly trained or pretrained spectrum predictor
  against reimplemented baselines on the same test split, and want one CSV table with
  average cosine similarity per model per dataset rather than scattered per-run metrics.
when_to_use_negative:
- Input is already a completed per-model metric table or you only need to compare
  published numbers without re-running predictions.
- You need spectrum-level or per-molecule similarity distributions rather than a single
  averaged value per model per dataset, since this skill collapses predictions into
  split-level averages.
- The evaluation metric must differ from vectorized cosine similarity over 0.1-Da
  binned vectors (e.g., you need SIS, MassSpecGym-native scoring, or spectral angle),
  as this skill is anchored to that one shared metric.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ICEBERG
  role: Spectrum predictor under evaluation; run prediction to generate binned spectra
    that populate its benchmark-table row
  repo: https://github.com/samgoldman97/ms-pred
- name: SCARF
  role: Baseline spectrum predictor run via run_scripts/scarf_model/05_predict_form_inten.py
    to fill its row of the benchmark table
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Baseline spectrum predictor run under run_scripts/cfm_id/ (not retrained;
    instructions for running are provided) to fill its row of the benchmark table
  repo: https://github.com/samgoldman97/ms-pred
- name: MassFormer
  role: Baseline spectrum predictor run via run_scripts/massformer_model/02_predict_massformer.py
    to fill its row of the benchmark table
  repo: https://github.com/Roestlab/massformer
- name: Python
  role: Runs the shared evaluation script analysis/spec_pred_eval.py and the per-model
    prediction scripts, then writes the aggregated CSV table
- name: uv
  role: Creates and activates the ms-pred environment (uv sync --extra cu124; source
    .venv/bin/activate) in which prediction and evaluation scripts run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:c746c204669af1e393f0993cd431e302c59dee42ab17ca6a3ad343169a1f8ee6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MassFormer
  - Python
  - SCARF
  - uv
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-table-compilation

## Summary

Assembles a per-model, per-dataset benchmark table of average cosine similarity values for ICEBERG and seven reimplemented baselines on NPLIB1 and NIST20 test splits, using a single shared vectorized cosine evaluation script so rows are directly comparable.

## When to use

Use when you must compare a newly trained or pretrained spectrum predictor against reimplemented baselines on the same test split, and want one CSV table with average cosine similarity per model per dataset rather than scattered per-run metrics.

## When NOT to use

- Input is already a completed per-model metric table or you only need to compare published numbers without re-running predictions.
- You need spectrum-level or per-molecule similarity distributions rather than a single averaged value per model per dataset, since this skill collapses predictions into split-level averages.
- The evaluation metric must differ from vectorized cosine similarity over 0.1-Da binned vectors (e.g., you need SIS, MassSpecGym-native scoring, or spectral angle), as this skill is anchored to that one shared metric.

## Inputs

- Predicted binned spectra (0.1-Da bins) for ICEBERG and each baseline model on the NPLIB1 and NIST20 test splits
- Ground-truth binned spectra for the NPLIB1 and NIST20 test splits
- Trained model checkpoints or pretrained weights for each of the eight models (ICEBERG, MassFormer, FixedVocab, SCARF, NEIMS-FFN, NEIMS-GNN, CFM-ID, and related retrained baselines)

## Outputs

- A single CSV benchmark table with one row per model and columns for average cosine similarity on NPLIB1 and NIST20
- Per-model average cosine similarity values (e.g., ICEBERG 0.627 on NPLIB1, 0.727 on NIST20)

## How to apply

Run ICEBERG prediction via src/ms_pred/iceberg/predict_smis.py and generate predicted binned spectra for each baseline with its own predict script (02_predict_ffn.py, 02_predict_gnn.py, 02_predict_molnetms.py equivalent, run_scripts/scarf_model/05_predict_form_inten.py, run_scripts/massformer_model/02_predict_massformer.py, and CFM-ID under run_scripts/cfm_id/), all using equivalent settings (same covariates, hyperparameter sweeps per model as the README specifies). Evaluate every model's predictions against ground truth with the same shared script, analysis/spec_pred_eval.py, which implements the vectorized cosine similarity over 0.1-Da binned vectors, rather than re-implementing the metric per model. Average the cosine similarity across the test split for each model on each dataset (NPLIB1, NIST20), then assemble the per-model, per-dataset averages into a single table and write it as a CSV. Validate by checking that the ICEBERG row reproduces the reported values (approximately 0.627 on NPLIB1 and 0.727 on NIST20, the latter nearly matching SCARF at 0.726) and that baseline rows fall within the reported ranges (e.g., MassFormer and FixedVocab each ~0.568 on NPLIB1, a 10% gap to ICEBERG) before the table is accepted.

## Related tools

- **ICEBERG** (Spectrum predictor under evaluation; run prediction to generate binned spectra that populate its benchmark-table row) — https://github.com/samgoldman97/ms-pred
- **SCARF** (Baseline spectrum predictor run via run_scripts/scarf_model/05_predict_form_inten.py to fill its row of the benchmark table) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Baseline spectrum predictor run under run_scripts/cfm_id/ (not retrained; instructions for running are provided) to fill its row of the benchmark table) — https://github.com/samgoldman97/ms-pred
- **MassFormer** (Baseline spectrum predictor run via run_scripts/massformer_model/02_predict_massformer.py to fill its row of the benchmark table) — https://github.com/Roestlab/massformer
- **Python** (Runs the shared evaluation script analysis/spec_pred_eval.py and the per-model prediction scripts, then writes the aggregated CSV table)
- **uv** (Creates and activates the ms-pred environment (uv sync --extra cu124; source .venv/bin/activate) in which prediction and evaluation scripts run)

## Examples

```
uv sync --extra cu124; source .venv/bin/activate; python src/ms_pred/iceberg/predict_smis.py --dataset nist20; python analysis/spec_pred_eval.py --pred-spec nist20_preds.tsv --binned 0.1 --metric cosine
```

## Evaluation signals

- ICEBERG row reproduces the reported values: average cosine similarity approximately 0.627 on NPLIB1 and 0.727 on NIST20.
- SCARF row on NIST20 lands at approximately 0.726, nearly equivalent to ICEBERG's 0.727.
- MassFormer and FixedVocab rows on NPLIB1 fall near 0.568 each, about 10% below ICEBERG's 0.627.
- All models are scored by the same shared script (analysis/spec_pred_eval.py) using vectorized cosine similarity over 0.1-Da binned vectors — confirm no per-model metric re-implementation was used.
- Output is a single CSV with one row per model and one column per dataset (NPLIB1, NIST20), each cell an average over the full test split.

## Limitations

- CFM-ID is not retrained by the ms-pred repo; only instructions for running it are provided, so its row may not reflect equivalent training settings.
- ICEBERG inference costs roughly 1 CPU second per molecule, nearly an order of magnitude more than SCARF, which slows full-split prediction for the ICEBERG row.
- The NPLIB1 dataset is being retired in the ms-pred main branch; reproducing its benchmark row requires checking out the legacy iceberg_1.0 branch.

## Evidence

- [methods] baseline cosine similarity ranges and shared metric: [evidence span withheld — anchor status: not_found]
- [readme] equivalent settings across models: "In order to fairly compare various spectra models, we implement a number of baselines and alternative models using equivalent settings across models (i.e., same covariates, hyperparameter sweeps for"
- [methods] benchmark row validation values: [evidence span withheld — anchor status: not_found]
- [readme] CFM-ID not retrained: "CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification](https://pubs.acs.org/doi/10.1021/acs.analchem.1c01465) (not retrained; instructions for running are provided)"
- [readme] NPLIB1 retirement caveat: "We are retiring the support of the NPLIB1 dataset (also referred as CANOPUS sometimes) in the main branch."
