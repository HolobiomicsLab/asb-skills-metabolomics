---
name: cross-validation-experiment-design
description: Use when fine-tuning a pre-trained retention-time model to a new chromatographic
  dataset and you need a stable estimate of R2 across folds and seeds rather than
  a single train/test split.
when_to_use_negative:
- You want to retrain the full model from scratch on new data (use train.py, not the
  freeze-and-fine-tune transfer protocol)
- Your data is a single small hold-out evaluation rather than a transfer-learning
  benchmark where cross-seed variance matters
- Your input is not molecular structures with retention times (e.g. it lacks InChI/RT
  columns and cannot be reshaped into the expected data.csv format)
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3174
tools:
- name: RT-Transformer
  role: Provides the pre-trained ResGAT + 1D-Transformer encoder whose Output-Block
    is re-initialized and fine-tuned under the repeated 10-fold CV protocol; run via
    transfer.py
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:96040e52743f579c98af26dee6c385645e3641e23f8f4e009a6f23d7562292ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-validation-experiment-design

## Summary

Designs a repeated k-fold cross-validation protocol for evaluating transfer-learned RT-Transformer retention-time models, in which the frozen ResGAT/1D-Transformer encoder is paired with a re-initialized Output-Block and performance is aggregated over 10 folds × 5 random seeds (50 runs per dataset) into an R2 distribution table.

## When to use

Use when fine-tuning a pre-trained retention-time model to a new chromatographic dataset and you need a stable estimate of R2 across folds and seeds rather than a single train/test split.

## When NOT to use

- You want to retrain the full model from scratch on new data (use train.py, not the freeze-and-fine-tune transfer protocol)
- Your data is a single small hold-out evaluation rather than a transfer-learning benchmark where cross-seed variance matters
- Your input is not molecular structures with retention times (e.g. it lacks InChI/RT columns and cannot be reshaped into the expected data.csv format)

## Inputs

- Chromatographic-method dataset as a CSV with 'InChI' and 'RT' columns (e.g. PredRet datasets)
- Pre-trained RT-Transformer checkpoint (best_state_dict.pth for full SMRT data or best_state_download_dict.pth for retained data)
- Number of folds (10) and random seeds (5) for the repeated CV design

## Outputs

- Per-dataset distribution of R2 scores across seeds and folds (e.g. r2_distribution.csv)
- Stability assessment per dataset (check that no R2 value falls below -0.5)

## How to apply

Prepare each dataset as a CSV with 'InChI' and 'RT' columns (the format the RT-Transformer repo expects for data.csv). Freeze the pre-trained ResGAT and 1D-Transformer encoder weights (choose best_state_dict.pth for the full-SMRT model or best_state_download_dict.pth for the retained-molecules model) and re-initialize only the Output-Block for the target chromatographic method. Run 10-fold cross-validation, repeating the whole procedure with 5 random seeds for 50 runs per dataset, and record the R2 score on every held-out fold. Aggregate fold-level R2 values into a per-dataset distribution (e.g. r2_distribution.csv) and check the stability criterion that no R2 falls below -0.5 for the full-SMRT pre-trained model; use distributions to compare against the retained-molecules model, especially where R2 exceeds 0.75.

## Related tools

- **RT-Transformer** (Provides the pre-trained ResGAT + 1D-Transformer encoder whose Output-Block is re-initialized and fine-tuned under the repeated 10-fold CV protocol; run via transfer.py) — https://github.com/01dadada/RT-Transformer

## Examples

```
cp best_state_dict.pth . && python transfer.py
```

## Evaluation signals

- All 50 runs (10 folds × 5 seeds) completed per dataset and each contributes one R2 value to the distribution table
- r2_distribution.csv contains a per-dataset R2 distribution with dataset/seed/fold provenance
- Stability check passes: no R2 score below -0.5 for the model pre-trained on the full SMRT dataset
- Comparative check: the full-SMRT pre-trained model's R2 distribution is tighter than the retained-molecules model's, and it wins when R2 > 0.75

## Limitations

- Stability of R2 distributions was demonstrated specifically for the full-SMRT pre-trained model; the retained-molecules model can produce less stable (lower) R2 values
- Performance depends on the target chromatographic method — results vary across the 41 external PredRet datasets
- The Output-Block must be re-initialized per dataset; fine-tuning the full encoder is not the validated protocol in the paper

## Evidence

- [other] Fine-tune the re-initialized Output-Block on each dataset with 10-fold cross-validation, repeated with five random seeds (50 runs per dataset).: [evidence span withheld — anchor status: not_found]
- [other] the model pre-trained on the complete SMRT dataset was more stable, with no R2 scores below -0.5: [evidence span withheld — anchor status: not_found]
- [readme] Prepare your dataset as a csv file which has "InChI" and "RT" columns.: [evidence span withheld — anchor status: external_doc]
- [readme] best_state_dict.pth The Best model of RT-Transformer train from full data.: [evidence span withheld — anchor status: external_doc]
- [abstract] The pre-trained RT-Transformer was further transferred to 5 datasets corresponding to different chromatographic conditions and fine-tuned.: "The pre-trained RT-Transformer was further transferred to 5 datasets corresponding to different chromatographic conditions and fine-tuned."
