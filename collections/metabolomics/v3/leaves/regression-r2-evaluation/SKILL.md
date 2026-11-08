---
name: regression-r2-evaluation
description: Use when comparing a fine-tuned regression model against a baseline across
  many small external datasets and you need a distribution of R2 scores (not a single
  point estimate) plus a stability criterion, as in RT-Transformer transfer to 41
  PredRet datasets.
when_to_use_negative:
- The task is classification (e.g., retained vs. not-retained molecule labeling) rather
  than continuous retention-time regression — R2 is not the appropriate metric.
- You need average absolute error on a single held-out test set (as in the SMRT evaluation
  reporting MAE of 27.30 / 33.41) rather than per-dataset R2 stability distributions
  across many small datasets.
- The dataset lacks continuous retention-time labels in the 'InChI'/'RT' CSV format
  expected by the transfer/evaluation scripts.
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: RT-Transformer
  role: Graph-attention + 1D-Transformer regression model whose per-fold R2 scores
    are computed and aggregated during transfer-learning evaluation
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:e59dd7f2f190c82341cc6d87981c2226d8555a980545dc79f99539c6e7550885
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# regression-r2-evaluation

## Summary

Evaluates a retention-time regression model (e.g., RT-Transformer) by aggregating R2 scores over repeated 10-fold cross-validation runs (multiple random seeds) per dataset, and judging stability via a floor criterion (no R2 below -0.5) plus comparisons at high-performance thresholds (R2 > 0.75).

## When to use

Use when comparing a fine-tuned regression model against a baseline across many small external datasets and you need a distribution of R2 scores (not a single point estimate) plus a stability criterion, as in RT-Transformer transfer to 41 PredRet datasets.

## When NOT to use

- The task is classification (e.g., retained vs. not-retained molecule labeling) rather than continuous retention-time regression — R2 is not the appropriate metric.
- You need average absolute error on a single held-out test set (as in the SMRT evaluation reporting MAE of 27.30 / 33.41) rather than per-dataset R2 stability distributions across many small datasets.
- The dataset lacks continuous retention-time labels in the 'InChI'/'RT' CSV format expected by the transfer/evaluation scripts.

## Inputs

- Per-fold R2 scores from 10-fold cross-validation repeated with 5 random seeds (50 runs per dataset)
- Predicted vs. true retention times for each held-out fold (CSV with 'InChI' and 'RT' columns as source data)
- 41 external PredRet chromatographic-method retention time datasets

## Outputs

- r2_distribution.csv — aggregated per-dataset R2 distribution table
- Stability verdict: whether any R2 score falls below -0.5 for the full-SMRT pre-trained model
- Comparison of full-SMRT vs retained-molecules models, particularly for datasets with R2 > 0.75

## How to apply

For each dataset, run 10-fold cross-validation repeated with five random seeds (50 runs per dataset), computing the R2 score on each held-out fold for every dataset/seed/fold combination. In the RT-Transformer transfer setting, freeze the pre-trained ResGAT and 1D-Transformer encoder weights and re-initialize only the Output-Block for fine-tuning on each PredRet chromatographic-method dataset. Aggregate the fold-level R2 values into a per-dataset distribution table (e.g., r2_distribution.csv). Apply the stability criterion from the study: verify that no R2 value falls below -0.5 for the model pre-trained on the full SMRT dataset (80,038 molecules). When comparing full-SMRT vs retained-molecules pre-trained checkpoints (best_state_dict.pth vs best_state_download_dict.pth), treat R2 > 0.75 as the regime where the full-SMRT model outperforms the retained-molecules model.

## Related tools

- **RT-Transformer** (Graph-attention + 1D-Transformer regression model whose per-fold R2 scores are computed and aggregated during transfer-learning evaluation) — https://github.com/01dadada/RT-Transformer

## Examples

```
python transfer.py  # fine-tune on data.csv (InChI/RT columns) using a downloaded pre-trained checkpoint, then aggregate per-fold R2 into r2_distribution.csv
```

## Evaluation signals

- r2_distribution.csv exists and contains one R2 distribution per external dataset (41 PredRet datasets) aggregated over 50 runs (10 folds × 5 seeds).
- Stability check passes: no R2 value falls below -0.5 for the model pre-trained on the full SMRT dataset.
- Comparison is stratified: full-SMRT model shows superiority specifically for datasets where R2 exceeds 0.75.
- Each dataset/seed/fold combination yields a recorded R2 score (e.g., no missing folds across the 50-run grid).
- Fine-tuning used the correct transfer setup: frozen ResGAT and 1D-Transformer encoder with a re-initialized Output-Block.

## Limitations

- Different chromatographic conditions yield different retention times for the same metabolite, so per-dataset R2 distributions (not pooled scores) are required — pooled metrics can mask instability on individual PredRet methods.
- The stability floor (R2 > -0.5) is a coarse criterion; datasets with low but above-floor R2 may still transfer poorly, and the full-SMRT model's advantage was specifically observed when R2 exceeded 0.75.
- The README does not document a changelog for the repository (01dadada/RT-Transformer), so checkpoint/behavior changes between versions cannot be traced from the repo alone.

## Evidence

- [abstract] In addition, RT-Transformer was applied to 41 external molecular retention time datasets. Extensive evaluations indicate that RT-Transformer has excellent scalability in predicting retention times: "In addition, RT-Transformer was applied to 41 external molecular retention time datasets. Extensive evaluations indicate that RT-Transformer has excellent scalability in predicting retention times"
- [readme] Prepare your dataset as a csv file which has "InChI" and "RT" columns.: [evidence span withheld — anchor status: external_doc]
- [readme] The SMRT dataset is collect from [this paper](https://doi.org/10.1038/s41467-019-13680-7): [evidence span withheld — anchor status: external_doc]
