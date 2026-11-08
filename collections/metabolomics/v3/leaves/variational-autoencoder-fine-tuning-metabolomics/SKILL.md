---
name: variational-autoencoder-fine-tuning-metabolomics
description: Use when a small labeled metabolomics cohort (e.g., binary classification,
  multi-class, or survival task) must be predicted and a pre-trained metabolomics
  VAE checkpoint exists to transfer from, outperforming randomly initialized or traditional
  ML baselines.
when_to_use_negative:
- The input is already raw LC/MS data (mzML files) — signal detection must first be
  done with mzLearn to produce the MS1 feature matrix
- No pre-trained checkpoint is available and the task is not a transfer-learning comparison
  — finetune_VAE_unsupervised_main with pretrained_dir=None reduces to random initialization
- The new data is from an LC mode other than HILIC positive mode — the current pre-trained
  model was developed specifically for HILIC positive mode
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: mzEmbed (mz_embed.finetune)
  role: Provides the fine-tuning entry points (finetune_VAE_unsupervised_main, best_finetune_model_test_eval)
    that load the pretrained checkpoint, unfreeze the final encoder layer and task
    head, and run Optuna search
  repo: github.com/ReviveMed/mzEmbed
- name: Optuna
  role: Hyperparameter optimization over latent_size, n_layers, kl_weight, dropout,
    and learning_rate during fine-tuning, with trial counts compared between transfer-learned
    and randomly initialized models
- name: mzLearn
  role: Upstream producer of the 2,736-feature MS1 ion-feature matrix that serves
    as the input representation for pretraining and fine-tuning
- name: VAE
  role: Generative model architecture pre-trained on the mzLearn feature matrix and
    fine-tuned for clinical outcome prediction on ccRCC data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:e5107650bf114d9da7491fce0a212d2f3b50eb6aaecef5e2dc3523420eee8bd9
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
  - mzEmbed (mz_embed.finetune)
  - mzLearn
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# variational-autoencoder-fine-tuning-metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes a VAE pre-trained on an mzLearn-derived 2,736-feature MS1 metabolomics matrix from 20,548 blood samples onto a new cohort (e.g., CheckMate025 ccRCC serum data), using transfer learning of the final encoder layer plus a task head, with Optuna hyperparameter search.

## When to use

Use when a small labeled metabolomics cohort (e.g., binary classification, multi-class, or survival task) must be predicted and a pre-trained metabolomics VAE checkpoint exists to transfer from, outperforming randomly initialized or traditional ML baselines.

## When NOT to use

- The input is already raw LC/MS data (mzML files) — signal detection must first be done with mzLearn to produce the MS1 feature matrix
- No pre-trained checkpoint is available and the task is not a transfer-learning comparison — finetune_VAE_unsupervised_main with pretrained_dir=None reduces to random initialization
- The new data is from an LC mode other than HILIC positive mode — the current pre-trained model was developed specifically for HILIC positive mode

## Inputs

- Fine-tune CSV feature matrices: X_Finetune_Train.csv, X_Finetune_Val.csv, X_Finetune_Test.csv (floats, z-scored, shape [n_samples, 2736], first column as row index)
- Label CSVs y_Finetune_* with task-specific columns (IMDC class, OS time/event)
- Pre-trained VAE checkpoint directory containing best_model.pt (trained on the 2,736-feature matrix from 20,548 blood samples)

## Outputs

- best_finetune_model.pt, study.html, and config.json written to the output directory
- Latent embeddings Z_* and exported encoder (via get_pretrain_encoder) for downstream analysis
- Task metrics dict from test evaluation, e.g. {'AUC': ...} for binary, {'F1': ...} for multiclass, {'C-index': ...} for survival
- Comparison table (e.g., table1_transfer_learning_comparison.csv) of transfer-learned vs randomly initialized vs traditional ML performance and Optuna trial counts

## How to apply

Load the mzLearn-derived MS1 feature matrix as z-scored float CSVs of shape [n_samples, 2736] (pandas.read_csv with index_col=0 semantics) and split into the reported train/validation/test partition (e.g., 543/149/149 for CheckMate025). Load the pre-trained VAE checkpoint via the mzEmbed codebase and freeze all weights except the final encoder layer and the prediction head, so transfer-learned representations are preserved while task-specific layers adapt. Run Optuna-driven fine-tuning (mz_embed.finetune.finetune_VAE_unsupervised_main with --pretrained_dir set) separately per cohort arm (e.g., ICI n=392 vs mTOR n=349), tuning latent_size, n_layers {2,3,4}, kl_weight, dropout (0–0.5), and learning_rate. Under identical splits and Optuna budget, train a randomly initialized VAE (same architecture, pretrained_dir=None) and traditional ML baselines for comparison. Evaluate with best_finetune_model_test_eval on the held-out test set using AUC (binary), F1 (3-class IMDC), or C-index (overall survival), and record the number of Optuna trials needed to reach the optimum — transfer learning reached its optimum in 38 vs 43 trials with lower validation loss in fewer epochs.

## Related tools

- **mzEmbed (mz_embed.finetune)** (Provides the fine-tuning entry points (finetune_VAE_unsupervised_main, best_finetune_model_test_eval) that load the pretrained checkpoint, unfreeze the final encoder layer and task head, and run Optuna search) — github.com/ReviveMed/mzEmbed
- **Optuna** (Hyperparameter optimization over latent_size, n_layers, kl_weight, dropout, and learning_rate during fine-tuning, with trial counts compared between transfer-learned and randomly initialized models)
- **mzLearn** (Upstream producer of the 2,736-feature MS1 ion-feature matrix that serves as the input representation for pretraining and fine-tuning)
- **VAE** (Generative model architecture pre-trained on the mzLearn feature matrix and fine-tuned for clinical outcome prediction on ccRCC data)

## Examples

```
python -m mz_embed.finetune.finetune_VAE_unsupervised_main --data_location /path/to/finetune_csvs --pretrained_dir /path/to/out_pretrain --output_dir /path/to/out_finetune; python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task binary
```

## Evaluation signals

- Transfer-learned fine-tuned models consistently outperform randomly initialized VAEs and traditional ML baselines under identical splits and Optuna budget
- The fine-tuned VAE reaches optimal Optuna hyperparameters in fewer trials (n=38 vs n=43) with lower validation loss in fewer epochs
- Test-set metrics match expected task types: AUC for binary IMDC, F1 for 3-class, C-index for overall survival (e.g., C-index around 0.67 cited in README example)
- Input CSVs must have exactly 2,736 float feature columns and load with index_col=0 semantics
- Fine-tuned model on ccRCC data matches or potentially outperforms IMDC criteria for survival prediction

## Limitations

- The current pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered
- Multi-modal learning for integrating various LC types and modes is not yet developed
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested
- The ICI-response classification lacks additional validation for clinical application
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments

## Evidence

- [results] Fine-tuned models with transfer learning consistently outperformed randomly initialized and traditional machine learning models: [evidence span withheld — anchor status: not_found]
- [readme] If pretrained_dir is set, weights are initialized from the pretrained model: [evidence span withheld — anchor status: external_doc]
- [readme] Task head evaluation on held-out test set: [evidence span withheld — anchor status: external_doc]
- [readme] Task metric options: [evidence span withheld — anchor status: external_doc]
