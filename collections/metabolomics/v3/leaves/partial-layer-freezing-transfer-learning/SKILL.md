---
name: partial-layer-freezing-transfer-learning
description: Use when a small labeled cohort (e.g. CheckMate025 ccRCC baseline serum
  metabolomics, 543/149/149 split) must be fine-tuned from the mzLearn 2,736-feature
  pre-trained VAE and naive full retraining would overfit or destroy pre-trained representations.
when_to_use_negative:
- No relevant pre-trained checkpoint exists (pretrained_dir=None) — the mzEmbed finetune
  entry point then randomly initializes weights, which is the control condition, not
  transfer learning
- Input is raw LC/MS mzML data rather than an mzLearn-derived feature matrix — signal
  detection/alignment must happen first
- Data are outside HILIC positive mode metabolomics — the current pre-trained model
  is specific to HILIC positive mode and other LC modes are not covered
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: mzEmbed
  role: Framework supplying the pre-trained VAE checkpoint and the finetune entry
    point (finetune_VAE_unsupervised_main with --pretrained_dir) that initializes
    weights from the pretrained model for transfer learning
  repo: github.com/ReviveMed/mzEmbed
- name: VAE
  role: Pre-trained generative model whose encoder is partially frozen; only the final
    encoder layer and prediction head are unfrozen for fine-tuning
  repo: github.com/ReviveMed/mzEmbed
- name: Optuna
  role: Hyperparameter search used under an identical budget for transfer-learned,
    randomly initialized, and traditional ML models so trial-efficiency (n=38 vs n=43)
    can be compared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:402257dd855d12f8f723761256240944e4ec494390850c792dfafb2354e13a37
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - Optuna
  - VAE
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# partial-layer-freezing-transfer-learning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tune a pre-trained metabolomics VAE by freezing all weights except the final encoder layer (and the task/prediction head), preserving the representations learned from ~20,548 public blood-sample spectra while adapting to a small unseen cohort.

## When to use

Use when a small labeled cohort (e.g. CheckMate025 ccRCC baseline serum metabolomics, 543/149/149 split) must be fine-tuned from the mzLearn 2,736-feature pre-trained VAE and naive full retraining would overfit or destroy pre-trained representations.

## When NOT to use

- No relevant pre-trained checkpoint exists (pretrained_dir=None) — the mzEmbed finetune entry point then randomly initializes weights, which is the control condition, not transfer learning
- Input is raw LC/MS mzML data rather than an mzLearn-derived feature matrix — signal detection/alignment must happen first
- Data are outside HILIC positive mode metabolomics — the current pre-trained model is specific to HILIC positive mode and other LC modes are not covered

## Inputs

- Pre-trained VAE checkpoint (best_model.pt) trained on the mzLearn MS1 feature matrix from ~20,548 blood samples (2,736 robust metabolite features, HILIC positive mode)
- Fine-tune CSV feature matrices: X_Finetune_Train.csv, X_Finetune_Val.csv, X_Finetune_Test.csv of shape [n_samples, 2736] (z-scored floats, index in first column)
- Label CSVs y_Finetune_* with task-specific columns (clinical outcome for ICI n=392 and mTOR n=349 arms; IMDC class or OS time/event)

## Outputs

- Fine-tuned model checkpoint best_finetune_model.pt plus study.html and config.json in the output directory
- Predictive performance metrics (e.g. AUC) and Optuna trial counts per model type (transfer-learned vs randomly initialized vs traditional ML)
- Comparison table table1_transfer_learning_comparison.csv

## How to apply

Load the pretrained VAE checkpoint from the mzEmbed pretraining run (Optuna-tuned on the 2,736-feature mzLearn matrix, e.g. latent_size=460, n_layers=3) and freeze every weight except the final encoder layer and the prediction head, so adaptation is confined to layers closest to the task. Run two fine-tuning arms separately (ICI n=392, mTOR n=349 on the CheckMate025 clinical-outcome task), using the identical train/validation/test split of 543/149/149 for all model types. Tune hyperparameters with Optuna under a fixed trial budget, and train control models — a randomly initialized VAE of the same architecture and traditional ML baselines — under identical splits and budget. Judge success by transfer-learned models consistently outperforming both controls (e.g. AUC), reaching their Optuna optimum in fewer trials (n=38 vs n=43), and attaining lower validation loss in fewer epochs; assemble the comparison into table1_transfer_learning_comparison.csv.

## Related tools

- **mzEmbed** (Framework supplying the pre-trained VAE checkpoint and the finetune entry point (finetune_VAE_unsupervised_main with --pretrained_dir) that initializes weights from the pretrained model for transfer learning) — github.com/ReviveMed/mzEmbed
- **VAE** (Pre-trained generative model whose encoder is partially frozen; only the final encoder layer and prediction head are unfrozen for fine-tuning) — github.com/ReviveMed/mzEmbed
- **Optuna** (Hyperparameter search used under an identical budget for transfer-learned, randomly initialized, and traditional ML models so trial-efficiency (n=38 vs n=43) can be compared)

## Examples

```
python -m mz_embed.finetune.finetune_VAE_unsupervised_main --data_location /path/to/finetune_csvs --pretrained_dir /path/to/out_pretrain --output_dir /path/to/out_finetune
```

## Evaluation signals

- Fine-tuned (transfer-learned) models consistently outperform both the randomly initialized VAE and traditional ML baselines on predictive performance (e.g. AUC) under identical splits
- Transfer-learned VAE reaches its optimal Optuna hyperparameters in fewer trials (n=38 vs n=43 for the random init)
- Lower validation loss is reached in fewer epochs than the randomly initialized control
- best_finetune_model.pt, study.html, and config.json are written to output_dir and held-out test evaluation returns metrics such as {'AUC': ...}
- Feature matrices conform to [n_samples, 2736] floats loadable with pandas.read_csv(..., index_col=0)

## Limitations

- The current pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested
- The ICI-response classification lacks additional validation for clinical application
- Fine-tuning (and pretraining) is Optuna-driven with early stopping monitored on validation loss, so results depend on the fixed trial budget

## Evidence

- [methods] task card workflow: [evidence span withheld — anchor status: not_found]
- [readme] README finetune entry: [evidence span withheld — anchor status: external_doc]
- [readme] README task-head refinement: [evidence span withheld — anchor status: external_doc]
