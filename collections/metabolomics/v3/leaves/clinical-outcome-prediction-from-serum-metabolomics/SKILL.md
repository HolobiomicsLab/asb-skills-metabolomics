---
name: clinical-outcome-prediction-from-serum-metabolomics
description: Use when you have a serum metabolomics feature matrix (mzLearn-derived
  MS1 peaks, e.g. 2,736 features) with clinical labels (IMDC class, OS time/event)
  and need to predict treatment response or survival in a new patient cohort.
when_to_use_negative:
- Input is raw LC/MS mzML files rather than an already-generated mzLearn feature table
  — use the mzLearn signal-detection skill first.
- Data comes from an LC mode or ionization mode other than HILIC positive — the pre-trained
  model is specific to that mode and transfer may be invalid.
- Goal is de novo compound identification or de-isotoping — mzLearn outputs are limited
  to the ion MS1 level and downstream annotation must use separate workflows.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: mzLearn
  role: Produces the normalized MS1 feature matrix that serves as the input to fine-tuning;
    ensures robust signal detection and drift correction across the cohort
- name: VAE (variational autoencoder)
  role: Pre-trained generative model whose encoder is fine-tuned on the new cohort's
    feature matrix for clinical outcome prediction
  repo: https://github.com/ReviveMed/mzEmbed
- name: Optuna
  role: Hyperparameter optimization framework used during fine-tuning to reach optimal
    settings in fewer trials than a randomly initialized baseline
- name: mzEmbed
  role: Codebase providing finetune_VAE_unsupervised_main and best_finetune_model_test_eval
    entry points for transfer learning and test-set evaluation
  repo: https://github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:7ba25d88337e2dcf60d9914b4772b60332cbe79c3f74c3592679aa5d5480cf40
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  repo_url: https://github.com/ReviveMed/mzEmbed
  tools:
  - Optuna
  - VAE
  - VAE (variational autoencoder)
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# clinical-outcome-prediction-from-serum-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes a pre-trained metabolomics variational autoencoder (VAE) on baseline serum LC/MS feature matrices to predict clinical outcomes such as ICI response and overall survival in renal cell carcinoma, outperforming randomly initialized and traditional ML baselines via transfer learning.

## When to use

Use when you have a serum metabolomics feature matrix (mzLearn-derived MS1 peaks, e.g. 2,736 features) with clinical labels (IMDC class, OS time/event) and need to predict treatment response or survival in a new patient cohort.

## When NOT to use

- Input is raw LC/MS mzML files rather than an already-generated mzLearn feature table — use the mzLearn signal-detection skill first.
- Data comes from an LC mode or ionization mode other than HILIC positive — the pre-trained model is specific to that mode and transfer may be invalid.
- Goal is de novo compound identification or de-isotoping — mzLearn outputs are limited to the ion MS1 level and downstream annotation must use separate workflows.

## Inputs

- CSV feature matrix of floats, shape [n_samples, 2736], derived from mzLearn MS1 signal detection (HILIC positive mode), z-scored
- Label CSVs with task-specific columns (IMDC class, OS time/event, ICI response status)
- Pre-trained VAE checkpoint (best_model.pt) from mzEmbed pretraining stage

## Outputs

- Fine-tuned model artifact (best_finetune_model.pt) plus Optuna study.html and config.json
- Test-set performance metrics dict, e.g. {'AUC': 0.93}, {'F1': 0.60}, or {'C-index': 0.67}
- Latent embeddings Z_Finetune_*.csv and SHAP feature-importance rankings of top predictive metabolite signals

## How to apply

Start from the pre-trained VAE developed on the mzLearn MS1 feature matrix from ~20,548 blood samples across 22 public studies. Partition the new cohort's feature matrix into train/validation/test splits (e.g. 543/149/149 for CheckMate025 baseline serum metabolomics). Fine-tune only the last encoder layer and prediction head using Optuna hyperparameter search (up to ~40 trials, max 200 epochs with early stopping on validation loss), run separately per clinical outcome/task (e.g. ICI arm n=392, mTOR arm n=349). Attach a lightweight task head for binary (AUC), multiclass (F1), or Cox survival (C-index) evaluation, and benchmark against a randomly initialized VAE and traditional ML baselines under identical splits and Optuna budget. Transfer learning reaches optimum in fewer trials (n=38 vs 43) with lower validation loss in fewer epochs.

## Related tools

- **mzLearn** (Produces the normalized MS1 feature matrix that serves as the input to fine-tuning; ensures robust signal detection and drift correction across the cohort)
- **VAE (variational autoencoder)** (Pre-trained generative model whose encoder is fine-tuned on the new cohort's feature matrix for clinical outcome prediction) — https://github.com/ReviveMed/mzEmbed
- **Optuna** (Hyperparameter optimization framework used during fine-tuning to reach optimal settings in fewer trials than a randomly initialized baseline)
- **mzEmbed** (Codebase providing finetune_VAE_unsupervised_main and best_finetune_model_test_eval entry points for transfer learning and test-set evaluation) — https://github.com/ReviveMed/mzEmbed

## Examples

```
python -m mz_embed.finetune.finetune_VAE_unsupervised_main --data_location /path/to/finetune_csvs --pretrained_dir /path/to/out_pretrain --output_dir /path/to/out_finetune
```

## Evaluation signals

- Test-set metric matches the expected task type: binary task returns an AUC value, multiclass returns F1, Cox returns C-index (README states e.g. {'AUC': 0.93} / {'F1': 0.60} / {'C-index': 0.67})
- Fine-tuned model with pretrained_dir initialization achieves lower validation loss in fewer epochs than a randomly initialized VAE trained under identical splits and Optuna budget
- Number of Optuna trials to reach the optimum is lower for the transfer-learned model (reported n=38 vs n=43 for the random baseline)
- Feature matrix passed to finetune matches the expected shape [n_samples, 2736] with first column as row index (pandas index_col=0 semantics)
- SHAP top-20 features are biologically plausible, e.g. enriched for acyl-carnitines (C6, C8, C12:1, C18:1) and LPC (C16:1) in the ICI-response model

## Limitations

- The pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered and multi-modal learning is not yet developed.
- ICI-response classification lacks additional validation for clinical application in treatment decision-making.
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested.
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments.

## Evidence

- [abstract] When fine-tuned on unseen renal cell carcinoma data, the pre-trained model improved risk stratification and overall survival predictions: "when fine-tuned on unseen renal cell carcinoma data, improved risk stratification and overall survival predictions"
- [readme] Fine-tuning entry point with optional transfer learning: [evidence span withheld — anchor status: external_doc]
- [readme] Test-set evaluation metrics per task type: [evidence span withheld — anchor status: external_doc]
- [full_text] Transfer learning converges in fewer trials than random initialization: [evidence span withheld — anchor status: not_found]
- [discussion] Feature-importance findings for ICI-response model: "The Top-20 feature list is enriched for acyl-carnitines (C6, C8, C12:1, C18:1) and LPC (C16:1)"
