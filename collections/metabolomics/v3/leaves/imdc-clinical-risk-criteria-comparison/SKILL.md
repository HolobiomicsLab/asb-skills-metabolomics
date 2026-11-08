---
name: imdc-clinical-risk-criteria-comparison
description: Use when a survival-prediction model (e.g., fine-tuned VAE on mzLearn
  MS1 feature matrices with OS time/event labels) must be validated against IMDC risk
  groups on held-out patient data.
when_to_use_negative:
- Input is raw LC/MS mzML data — run mzLearn signal detection and VAE fine-tuning
  first; this skill starts from an existing feature matrix
- The cohort lacks IMDC clinical risk annotations or OS time/event labels, making
  the head-to-head comparison impossible
- The pre-trained model is not HILIC positive mode — the article's representations
  are specific to that LC mode
edam_operation: http://edamontology.org/operation_2945
tools:
- name: mzEmbed
  role: Provides the fine-tuned VAE encoder and task-head evaluation used to score
    test patients for risk stratification and survival prediction
  repo: github.com/ReviveMed/mzEmbed
- name: VAE
  role: Pre-trained generative model fine-tuned on ccRCC baseline serum metabolomics
    to produce risk scores for stratification
  repo: github.com/ReviveMed/mzEmbed
- name: SHAP
  role: Feature-importance analysis on the fine-tuned models to derive Top-20 metabolite
    lists supporting the stratification comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:692674eec1b5032a9844e1b0e1d3e547abfb14a7231c4e8c7843c5166c866044
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# imdc-clinical-risk-criteria-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a fine-tuned metabolomics VAE's patient stratification against the clinically defined IMDC risk groups on held-out ccRCC data, using Kaplan-Meier curves, log-rank tests, and hazard ratios on overall survival. This tests whether metabolite-based latent representations can match or outperform the gold-standard clinical criteria.

## When to use

Use when a survival-prediction model (e.g., fine-tuned VAE on mzLearn MS1 feature matrices with OS time/event labels) must be validated against IMDC risk groups on held-out patient data.

## When NOT to use

- Input is raw LC/MS mzML data — run mzLearn signal detection and VAE fine-tuning first; this skill starts from an existing feature matrix
- The cohort lacks IMDC clinical risk annotations or OS time/event labels, making the head-to-head comparison impossible
- The pre-trained model is not HILIC positive mode — the article's representations are specific to that LC mode

## Inputs

- Held-out test-cohort mzLearn MS1 metabolite feature matrix (n=143 ccRCC patients, z-scored floats, shape [n_samples, 2736])
- Overall-survival labels (OS time and event) for the test cohort
- IMDC risk-group assignments (poor/intermediate/favorable) per patient
- Fine-tuned prognostic VAE model checkpoint (best_finetune_model.pt) and saved config

## Outputs

- Kaplan-Meier OS curves for VAE-defined and IMDC-defined risk groups
- Log-rank test p-values and hazard ratios for each stratification
- Predictive-model log-rank p-values per treatment arm (ICI, mTOR)
- fig5_validation_results.csv with log-rank p-values and stratification assignments

## How to apply

Score each held-out test patient with the fine-tuned prognostic VAE and assign VAE-defined risk groups from the predicted risk scores. Run Kaplan-Meier estimation and log-rank tests on the VAE-defined groups and, in parallel, on the IMDC-defined groups (poor/intermediate/favorable), then compare log-rank p-values and hazard ratios. Reproduction from this paper used a ccRCC held-out test cohort of n=143 patients with overall-survival labels; validation succeeded when all VAE risk groups had significantly distinct OS distributions with improved hazard ratios, while IMDC achieved significance only for poor vs favorable. For the paired predictive task, score the same cohort under ICI and mTOR treatment arms and require a significant ICI log-rank p-value (reported 2.1e-3) with a non-significant mTOR arm (reported 0.61) to demonstrate treatment-predictive rather than purely prognostic stratification.

## Related tools

- **mzEmbed** (Provides the fine-tuned VAE encoder and task-head evaluation used to score test patients for risk stratification and survival prediction) — github.com/ReviveMed/mzEmbed
- **VAE** (Pre-trained generative model fine-tuned on ccRCC baseline serum metabolomics to produce risk scores for stratification) — github.com/ReviveMed/mzEmbed
- **SHAP** (Feature-importance analysis on the fine-tuned models to derive Top-20 metabolite lists supporting the stratification comparison)

## Examples

```
python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task cox
```

## Evaluation signals

- Predictive-model ICI-arm log-rank p-value is significant (reported 2.1e-3) while the mTOR arm is not (reported 0.61)
- All VAE-defined risk groups show significantly distinct OS distributions on the n=143 held-out test set, with improved hazard ratios vs IMDC
- IMDC comparison shows significance only for poor vs favorable groups, confirming the VAE matches or outperforms IMDC
- SHAP Top-20 lists from the prognostic and predictive models share 3 of 20 features including acyl-carnitines (C6, C8, C12:1, C18:1) and LPC C16:1
- fig5_validation_results.csv contains the log-rank p-values and stratification assignments for both stratifications

## Limitations

- The ICI-response classification lacks additional validation for clinical application
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested
- mzLearn outputs are limited to ion MS1 level; downstream de-isotoping, annotation, and MS/MS confirmation are needed to interpret the lipid and carnitine signals driving stratification

## Evidence

- [readme] README: supported fine-tuning tasks including survival analysis: [evidence span withheld — anchor status: external_doc]
- [readme] README: held-out test evaluation metrics per task: [evidence span withheld — anchor status: not_found]
