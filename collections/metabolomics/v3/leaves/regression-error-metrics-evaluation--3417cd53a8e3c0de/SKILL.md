---
name: regression-error-metrics-evaluation
description: Use when evaluating a retention time prediction model's test-set performance
  in seconds against experimental LC retention times, e.g. SMRT benchmark splits or
  transfer-learning datasets.
when_to_use_negative:
- Classification-style evaluation (e.g. retained vs not-retained labels) or ranking
  metrics — this skill only covers absolute-error regression metrics in time units
- Evaluating fine-tuned transfer-learning performance on external chromatographic-condition
  datasets where the paper reports competitiveness against state-of-the-art methods
  rather than fixed MAE/MedAE thresholds
- Assessing model training convergence (loss curves) rather than final held-out test
  error
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: RT-Transformer
  role: Model under evaluation whose predicted retention times are compared with experimental
    values to compute MAE and MedAE; also supplies benchmark splits and pretrained
    checkpoints
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:21336a26c09c1deb979730750125a3e4ca5ad0eed57db74ed07534599b7c6b5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# regression-error-metrics-evaluation

## Summary

Computes mean absolute error (MAE) and median absolute error (MedAE) between predicted and experimental liquid chromatography retention times, the primary metrics used by RT-Transformer to benchmark retention time prediction on the SMRT dataset.

## When to use

Use when evaluating a retention time prediction model's test-set performance in seconds against experimental LC retention times, e.g. SMRT benchmark splits or transfer-learning datasets.

## When NOT to use

- Classification-style evaluation (e.g. retained vs not-retained labels) or ranking metrics — this skill only covers absolute-error regression metrics in time units
- Evaluating fine-tuned transfer-learning performance on external chromatographic-condition datasets where the paper reports competitiveness against state-of-the-art methods rather than fixed MAE/MedAE thresholds
- Assessing model training convergence (loss curves) rather than final held-out test error

## Inputs

- Trained RT-Transformer model checkpoint (e.g. best_state_dict.pth or best_state_download_dict.pth from the RT-Transformer repo)
- Held-out SMRT test partition with experimental retention times in seconds, either the full 80,038-molecule dataset or the retained subset with RT > 300 s
- CSV of per-molecule predicted retention times from model inference

## Outputs

- Mean absolute error (MAE) in seconds on the test set
- Median absolute error (MedAE) in seconds on the test set
- Summary metrics file (e.g. rt_transformer_smrt_test_metrics.json) and per-molecule prediction CSV (e.g. rt_transformer_all_molecules_predictions.csv)

## How to apply

After running inference with the trained RT-Transformer (graph attention network coupled with a 1D-Transformer) on the held-out SMRT test partition, collect per-molecule predicted and measured retention times in seconds and compute MAE = mean(|ŷ − y|) and MedAE = median(|ŷ − y|). The article reports two evaluation conditions that must be kept separate: the retained-molecules condition (training and evaluation on molecules with retention times > 300 s, not retained molecules removed) yielding MAE 27.30 s and MedAE 12.46 s, and the no-samples-removed condition over all 80,038 SMRT molecules yielding MAE 33.41 s and MedAE 13.05 s. When comparing against published numbers, match the condition and the same train/test split from the RT-Transformer repository; MedAE should be substantially lower than MAE because the error distribution is right-skewed. Write the summary metrics to a JSON file (e.g. rt_transformer_smrt_test_metrics.json) and per-molecule predictions to CSV for auditability.

## Related tools

- **RT-Transformer** (Model under evaluation whose predicted retention times are compared with experimental values to compute MAE and MedAE; also supplies benchmark splits and pretrained checkpoints) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py
```

## Evaluation signals

- MAE and MedAE are reported in seconds and match the condition-specific reference values: 27.30 s / 12.46 s for the retained (RT > 300 s) molecules and 33.41 s / 13.05 s for the no-samples-removed condition on SMRT
- MedAE is lower than MAE, consistent with a right-skewed absolute-error distribution over molecules
- The evaluation is performed on the held-out SMRT test split only, not on training data, and the split matches the RT-Transformer benchmark partition
- Summary metrics are persisted to a machine-readable JSON alongside per-molecule predictions for reproducibility

## Limitations

- Reported MAE values differ markedly by filtering condition (27.30 s vs 33.41 s), so metrics are only comparable when the same molecule-filtering rule is applied
- The 27.30 s figure reflects training on retained molecules only (RT > 300 s), which excludes not-retained molecules and narrows the evaluated retention range
- Metrics are computed per dataset/chromatographic condition; performance on transfer-learning datasets from PredRet may differ from the SMRT benchmark
- MAE and MedAE alone do not capture tail behaviour; the article reports average and median absolute error but no additional uncertainty or per-class breakdown

## Evidence

- [abstract] Abstract quote on MAE under two filtering conditions: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [abstract] Abstract quote describing the pre-training dataset scale: "by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [readme] README quote naming the pretrained checkpoint files: [evidence span withheld — anchor status: external_doc]
- [readme] README quote describing model input architecture: [evidence span withheld — anchor status: external_doc]
