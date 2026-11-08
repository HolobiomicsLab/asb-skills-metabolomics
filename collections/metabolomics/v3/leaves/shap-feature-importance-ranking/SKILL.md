---
name: shap-feature-importance-ranking
description: Use when a trained or fine-tuned metabolomics model (e.g. VAE on a 2,736-feature
  mzLearn matrix) needs interpretation, and top-ranked ion features must be identified
  for biological annotation or biomarker shortlisting.
when_to_use_negative:
- Input is raw LC/MS mzML data rather than an already-generated mzLearn feature matrix
  — SHAP ranking operates on model features, not signal detection
- Model is not yet fine-tuned or lacks a task head; run `finetune_VAE_unsupervised_main`
  and `best_finetune_model_test_eval` first
- Goal is definitive compound identification — mzLearn features are ion MS1 level
  and need separate de-isotoping/MS/MS annotation workflows
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzEmbed
  role: Provides the fine-tuned VAE model artifacts and evaluation workflow whose
    predictions are decomposed with SHAP
  repo: github.com/ReviveMed/mzEmbed
- name: mzLearn
  role: Generates the 2,736-feature MS1 ion matrix that defines the feature space
    SHAP ranks over
- name: SHAP
  role: Computes per-ion-feature importance values and derives the Top-20 ranked lists
    on fine-tuned models
- name: VAE
  role: The fine-tuned generative model whose latent and input features are attributed
    by SHAP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:983d52afd34c2b8782289463d9a9ceb2903f9f3eb370769a75fb38efd6d6c6e7
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
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# shap-feature-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies SHAP feature-importance analysis to fine-tuned VAE models built on mzLearn MS1 metabolite feature matrices, ranking ion features (e.g. acyl-carnitines, LPC) that drive survival or treatment-response predictions in renal cell carcinoma.

## When to use

Use when a trained or fine-tuned metabolomics model (e.g. VAE on a 2,736-feature mzLearn matrix) needs interpretation, and top-ranked ion features must be identified for biological annotation or biomarker shortlisting.

## When NOT to use

- Input is raw LC/MS mzML data rather than an already-generated mzLearn feature matrix — SHAP ranking operates on model features, not signal detection
- Model is not yet fine-tuned or lacks a task head; run `finetune_VAE_unsupervised_main` and `best_finetune_model_test_eval` first
- Goal is definitive compound identification — mzLearn features are ion MS1 level and need separate de-isotoping/MS/MS annotation workflows

## Inputs

- Fine-tuned VAE model artifacts from mzEmbed (e.g. `best_finetune_model.pt` with saved config)
- Feature matrix CSV of floats shaped [n_samples, 2736] (mzLearn robust peak set, z-scored)
- Task labels CSV (IMDC class, OS time/event, or treatment-response class)

## Outputs

- Per-feature SHAP importance values ranked across all 2,736 ion features
- Top-20 ranked ion-feature list per model (e.g. prognostic vs predictive VAE)
- Annotated metabolite classes for top features (e.g. acyl-carnitines C6/C8/C12:1/C18:1, LPC C16:1)
- Overlap count between Top-20 lists across models (expected 3/20 in the reported ccRCC analysis)

## How to apply

After fine-tuning a VAE (via mzEmbed's finetune workflow) on patient metabolomics data — e.g. ccRCC baseline serum with OS or ICI/mTOR response labels — attach a task head (binary IMDC, 3-class F1, or Cox) using `best_finetune_model_test_eval` and compute SHAP values over the model's input features to rank metabolite ion signals by contribution to prediction. In this article, SHAP was run on both a prognostic and a predictive (ICI/mTOR) fine-tuned model, each yielding a Top-20 ranked list of ion features. Rank lists are then mapped to annotated metabolite classes: the ICI-response model's Top-20 was enriched for acyl-carnitines (C6, C8, C12:1, C18:1) and LPC (C16:1), with 3 of 20 features overlapping between prognostic and predictive lists. Because mzLearn outputs are ion-level MS1 features (median rt/m/z), ranked features require downstream de-isotoping, annotation, and MS/MS confirmation before biological interpretation is definitive.

## Related tools

- **mzEmbed** (Provides the fine-tuned VAE model artifacts and evaluation workflow whose predictions are decomposed with SHAP) — github.com/ReviveMed/mzEmbed
- **mzLearn** (Generates the 2,736-feature MS1 ion matrix that defines the feature space SHAP ranks over)
- **SHAP** (Computes per-ion-feature importance values and derives the Top-20 ranked lists on fine-tuned models)
- **VAE** (The fine-tuned generative model whose latent and input features are attributed by SHAP)

## Examples

```
python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task cox
```

## Evaluation signals

- SHAP Top-20 lists from prognostic and predictive models share 3 of 20 features, including reported lipid and carnitine members
- Top-ranked features map to biologically plausible classes: acyl-carnitines (C6, C8, C12:1, C18:1) and LPC (C16:1) in the ICI-response model
- Ranked features are within the 2,736-column mzLearn robust peak set (shape [n_samples, 2736])
- Task-head metrics on the same model are reasonable (AUC ~0.93 binary IMDC, F1 ~0.60 3-class, C-index ~0.67 OS), confirming the model being explained is competent
- SHAP importance distributions are non-degenerate (no single feature dominates the full 2,736-feature ranking)

## Limitations

- mzLearn outputs are ion MS1 level; ranked features lack de-isotoping and MS/MS identification, so annotation requires downstream workflows
- The pre-trained model is specific to HILIC positive mode, limiting transferability of ranked features to other LC modes
- The ICI-response classification lacks additional validation for clinical application
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments, which could confound SHAP attributions
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested

## Evidence

- [readme] README: expected input shape [n_samples, 2736]: [evidence span withheld — anchor status: external_doc]
- [readme] README: task-head evaluation entry point: [evidence span withheld — anchor status: external_doc]
