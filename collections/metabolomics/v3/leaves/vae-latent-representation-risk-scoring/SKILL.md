---
name: vae-latent-representation-risk-scoring
description: Use when you have an mzLearn-generated MS1 ion-feature matrix for a patient
  cohort plus survival (OS time/event), IMDC risk class, or treatment-response labels
  and need VAE-latent-based prognostic/predictive stratification.
when_to_use_negative:
- Input data is raw mzML files rather than an mzLearn-derived feature table — run
  mzLearn signal detection first; the VAE expects the 2,736-feature MS1 matrix
- Data was acquired outside HILIC positive mode LC-MS — the pre-trained model was
  developed specifically for HILIC positive mode and other LC modes/modalities are
  not covered
- De-isotoping or MS/MS-level compound identification is the goal — mzLearn outputs
  are ion MS1 level only and downstream mapping/annotation workflows must be applied
  separately
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3172
tools:
- name: mzLearn
  role: Produces the zero-parameter, drift-corrected MS1 ion-feature matrix that serves
    as the VAE's input representation space
  repo: github.com/ReviveMed/mzEmbed
- name: VAE
  role: Pre-trained generative model over the mzLearn feature matrix whose encoder
    latents are fine-tuned and re-headed for risk scoring and survival prediction
  repo: github.com/ReviveMed/mzEmbed
- name: SHAP
  role: Feature-importance analysis on the fine-tuned models to rank Top-20 metabolite
    signals (acyl-carnitines C6/C8/C12:1/C18:1, LPC C16:1) for biological validation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:24730e0ae6b6bdcc578526ba914b40ee8848fe84e374336b6afb5249195458fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - SHAP
  - VAE
  - mzLearn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# vae-latent-representation-risk-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes a VAE pre-trained on mzLearn MS1 feature matrices (2,736 robust metabolite features from 20,548 blood samples) on task-specific clinical data (e.g., ccRCC serum metabolomics) to produce latent representations used for risk stratification, overall-survival prediction, and treatment-response classification that are benchmarked against IMDC criteria.

## When to use

Use when you have an mzLearn-generated MS1 ion-feature matrix for a patient cohort plus survival (OS time/event), IMDC risk class, or treatment-response labels and need VAE-latent-based prognostic/predictive stratification.

## When NOT to use

- Input data is raw mzML files rather than an mzLearn-derived feature table — run mzLearn signal detection first; the VAE expects the 2,736-feature MS1 matrix
- Data was acquired outside HILIC positive mode LC-MS — the pre-trained model was developed specifically for HILIC positive mode and other LC modes/modalities are not covered
- De-isotoping or MS/MS-level compound identification is the goal — mzLearn outputs are ion MS1 level only and downstream mapping/annotation workflows must be applied separately

## Inputs

- CSV feature matrices of mzLearn-detected MS1 ion intensities, shape [n_samples, 2736], z-scored floats, first column as row index (X_Pretrain_* and X_Finetune_{Train,Val,Test}.csv)
- Label CSVs with task-specific columns: IMDC class, overall-survival time/event, or treatment-response/arm labels
- Pre-trained VAE checkpoint directory containing best_model.pt and saved config

## Outputs

- Fine-tuned VAE model artifacts (best_finetune_model.pt, study.html, config.json)
- Latent embeddings Z_*_*.csv per split (shape [n, latent_size]) plus exported encoder.pt
- Task metrics dict from held-out test evaluation (AUC for binary IMDC, F1 for 3-class IMDC, C-index for OS)
- VAE-defined risk-group assignments with Kaplan-Meier curves, log-rank p-values, and hazard ratios vs IMDC stratification
- SHAP Top-20 ranked metabolite feature lists per fine-tuned model

## How to apply

First pre-train the VAE on the standardized 2,736-feature mzLearn matrix (e.g., latent_size=460, 3 layers, Optuna-driven tuning with validation-loss early stopping), then export the encoder with get_pretrain_encoder. Fine-tune on the task cohort by initializing from the pre-trained weights (transfer learning) and retraining the last layer for the target task — binary/multiclass IMDC classification or Cox survival analysis. For risk scoring, score held-out patients with the fine-tuned prognostic VAE to assign VAE-defined risk groups, then compare Kaplan-Meier OS distributions and log-rank significance against IMDC risk groups; in the ccRCC validation (n=143) the prognostic VAE gave significantly distinct OS distributions for all risk groups while IMDC was significant only for poor vs favorable. For predictive modeling, stratify by predicted response within treatment arms (ICI vs mTOR) and require arm-specific log-rank significance. Finally run SHAP on the fine-tuned model to extract Top-20 metabolite features and check for biologically plausible lipid/carnitine signals shared across tasks.

## Related tools

- **mzLearn** (Produces the zero-parameter, drift-corrected MS1 ion-feature matrix that serves as the VAE's input representation space) — github.com/ReviveMed/mzEmbed
- **VAE** (Pre-trained generative model over the mzLearn feature matrix whose encoder latents are fine-tuned and re-headed for risk scoring and survival prediction) — github.com/ReviveMed/mzEmbed
- **SHAP** (Feature-importance analysis on the fine-tuned models to rank Top-20 metabolite signals (acyl-carnitines C6/C8/C12:1/C18:1, LPC C16:1) for biological validation)

## Examples

```
python -m mz_embed.finetune.finetune_VAE_unsupervised_main --data_location /path/to/finetune_csvs --pretrained_dir /path/to/out_pretrain --output_dir /path/to/out_finetune && python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task cox
```

## Evaluation signals

- Held-out test metrics within reported ranges: binary IMDC AUC ~0.93, 3-class IMDC F1 ~0.60, OS C-index ~0.67
- On the n=143 ccRCC test cohort, the prognostic VAE yields log-rank-significant OS separation for ALL predicted risk groups, whereas IMDC is significant only for poor vs favorable
- Predictive-model validation: ICI-arm log-rank p-value significant (reported 2.1e-3) while mTOR arm is not (reported 0.61)
- SHAP Top-20 lists from the prognostic and predictive models share 3 of 20 features, including the reported acyl-carnitine (C6, C8, C12:1, C18:1) and LPC (C16:1) members
- Schema checks: input CSVs parse as [n_samples, 2736] floats with index_col=0; exported latent embeddings have shape [n, latent_size]

## Limitations

- The current pre-trained model was developed specifically for HILIC positive mode; other LC modes and modalities are not covered, and multi-modal learning for integrating various LC types is not yet developed
- The ICI-response classification lacks additional validation for clinical application
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments
- mzLearn outputs are limited to the ion MS1 level; no de-isotoping or MS/MS identification is performed and no MS/MS integration into pretraining is implemented

## Evidence

- [readme] Supports fine-tuning for binary classification, multi-class classification, and survival analysis.: [evidence span withheld — anchor status: external_doc]
- [readme] task: {'binary', 'multiclass', 'cox'} — IMDC binary (AUC), IMDC 3-class (F1), or OS (C-index).: [evidence span withheld — anchor status: not_found]
