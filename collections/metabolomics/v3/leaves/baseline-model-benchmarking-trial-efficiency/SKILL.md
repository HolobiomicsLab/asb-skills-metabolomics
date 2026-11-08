---
name: baseline-model-benchmarking-trial-efficiency
description: Use when evaluating whether transfer learning from a pre-trained metabolomics
  VAE (e.g. 2,736-feature mzLearn matrix) improves downstream prediction and Optuna
  search efficiency versus random initialization or traditional ML.
when_to_use_negative:
- Input data is already a small traditional-ML-friendly feature table with no pre-trained
  checkpoint available for transfer learning — direct ML baselines alone suffice.
- The task is unsupervised pretraining of the VAE itself rather than benchmarking
  fine-tuned models on a downstream clinical outcome.
- Data are not in HILIC positive mode or the pre-trained model was not built from
  the 2,736-feature mzLearn robust peak set, since the pre-trained representations
  are mode-specific.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: mzLearn
  role: Generates the MS1 feature matrix (2,736 robust metabolite signals) from raw
    mzML files that serves as the shared input for all benchmarked models
- name: VAE
  role: The transfer-learned and randomly-initialized model being benchmarked for
    clinical outcome prediction on the mzLearn feature matrix
- name: Optuna
  role: Hyperparameter search framework used with an identical budget across all compared
    models so trial counts to reach the optimum are directly comparable
- name: mzEmbed
  role: Codebase providing finetune_VAE_unsupervised_main (transfer or random init)
    and best_finetune_model_test_eval for evaluating AUC/F1/C-index on the held-out
    test set
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:1c5beada3ba533a21152a4633b12bda2e3423adc476f70d9eb5ea2a0986fc9d3
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Baseline model benchmarking and trial-efficiency comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a transfer-learned VAE fine-tuned on baseline serum metabolomics of ccRCC patients (CheckMate025, ICI vs mTOR arms) against a randomly initialized VAE and traditional ML baselines under identical splits and Optuna budgets, quantifying both predictive performance and the number of Optuna trials needed to reach the optimum.

## When to use

Use when evaluating whether transfer learning from a pre-trained metabolomics VAE (e.g. 2,736-feature mzLearn matrix) improves downstream prediction and Optuna search efficiency versus random initialization or traditional ML.

## When NOT to use

- Input data is already a small traditional-ML-friendly feature table with no pre-trained checkpoint available for transfer learning — direct ML baselines alone suffice.
- The task is unsupervised pretraining of the VAE itself rather than benchmarking fine-tuned models on a downstream clinical outcome.
- Data are not in HILIC positive mode or the pre-trained model was not built from the 2,736-feature mzLearn robust peak set, since the pre-trained representations are mode-specific.

## Inputs

- Baseline serum metabolomics MS1 feature matrix from mzLearn for CheckMate025 (CSV floats, shape [n_samples, 2736])
- Train/validation/test split indices or label files (X_Finetune_Train/Val/Test.csv and y_Finetune_* for IMDC class, OS time/event)
- Pre-trained VAE checkpoint directory containing best_model.pt (mzEmbed pretrain output)
- Traditional ML baseline configurations

## Outputs

- Per-model predictive performance metrics (AUC for binary IMDC, F1 for 3-class, C-index for OS)
- Per-model Optuna trial count to reach the optimum (transfer-learned vs random-init, e.g. n=38 vs n=43)
- Validation-loss-versus-epoch curves for each model type
- table1_transfer_learning_comparison.csv (or equivalent comparison table)

## How to apply

Load the mzLearn-derived baseline serum metabolomics MS1 feature matrix for CheckMate025 and partition samples into the reported 543/149/149 train/validation/test split. Load the pre-trained VAE checkpoint (trained on the 2,736-feature mzLearn matrix from 20,548 blood samples) and fine-tune it with Optuna hyperparameter search separately for the ICI arm (n=392) and the mTOR arm (n=349) on the clinical outcome prediction task. Under identical splits and the same Optuna budget, train control models: a randomly initialized VAE of the same architecture fine-tuned the same way, and traditional ML baselines. Record predictive performance (e.g. AUC for the IMDC binary task, C-index for overall survival via the cox task) and the number of Optuna trials required to reach the optimum for each model type. Assemble the comparison, verifying that transfer-learned models consistently outperform both baselines and reach their optimum in fewer trials (n=38 vs n=43) with lower validation loss in fewer epochs. Write the results to a comparison table (e.g. table1_transfer_learning_comparison.csv).

## Related tools

- **mzLearn** (Generates the MS1 feature matrix (2,736 robust metabolite signals) from raw mzML files that serves as the shared input for all benchmarked models)
- **VAE** (The transfer-learned and randomly-initialized model being benchmarked for clinical outcome prediction on the mzLearn feature matrix)
- **Optuna** (Hyperparameter search framework used with an identical budget across all compared models so trial counts to reach the optimum are directly comparable)
- **mzEmbed** (Codebase providing finetune_VAE_unsupervised_main (transfer or random init) and best_finetune_model_test_eval for evaluating AUC/F1/C-index on the held-out test set) — github.com/ReviveMed/mzEmbed

## Examples

```
python -m mz_embed.finetune.finetune_VAE_unsupervised_main --data_location /path/to/finetune_csvs --pretrained_dir /path/to/out_pretrain --output_dir /path/to/out_finetune; python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task binary
```

## Evaluation signals

- Transfer-learned models consistently outperform randomly initialized and traditional ML baselines on the identical test split
- Optuna trial count to reach the optimum is lower for the fine-tuned VAE (n=38) than the random-init control (n=43)
- Validation loss for the transfer-learned model is lower in fewer epochs than for the randomly initialized model
- Feature-matrix shape is exactly [n_samples, 2736] floats with index_col=0 semantics for every input CSV
- Per-task metrics fall within expected ranges from the README (AUC ~0.93 for binary IMDC, F1 ~0.60 for 3-class, C-index ~0.67 for OS)

## Limitations

- The pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered, limiting benchmark generalizability across data types.
- Iterative mzLearn preprocessing can take up to 2.5 days on a single machine, and no parallel processing on multiple compute nodes is implemented.
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, which affects comparability of the underlying mzLearn feature matrices.
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested; benchmark results may not hold for new cohorts.
- Batch covariates or domain-invariant regularization are not yet incorporated, which could confound cross-model comparisons on large-scale deployments.

## Evidence

- [other] Fine-tuned models with transfer learning consistently outperformed randomly initialized and traditional machine learning models, and the fine-tuned VAE reached optimal Optuna hyperparameters in fewer trials (n=38 vs n=43): [evidence span withheld — anchor status: not_found]
- [readme] Transfer or random initialization of fine-tuned VAE: [evidence span withheld — anchor status: external_doc]
- [readme] IMDC binary task evaluated by AUC on held-out test set: [evidence span withheld — anchor status: external_doc]
- [readme] Feature matrix shape expectation: [evidence span withheld — anchor status: external_doc]
