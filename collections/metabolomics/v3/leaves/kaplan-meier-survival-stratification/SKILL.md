---
name: kaplan-meier-survival-stratification
description: Use when patient-level metadata includes overall survival time/event
  and predicted risk groups (e.g. VAE embeddings or IMDC classes) and you must test
  whether groups separate by OS via KM curves and log-rank p-values.
when_to_use_negative:
- Input feature tables are not patient-level with survival time/event metadata — KM
  and log-rank tests are undefined without OS labels
- The cohort is too small to estimate KM curves with meaningful follow-up; the paper's
  held-out test set was n=143 ccRCC patients
- The question is feature-level metabolite importance rather than patient-level survival
  separation (use SHAP analysis instead)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: mzEmbed
  role: Provides the fine-tuned VAE encoder whose latent predictions define the risk
    groups tested with KM/log-rank; test evaluation entry point returns metrics on
    held-out data
  repo: https://github.com/ReviveMed/mzEmbed
- name: VAE
  role: Supplies the pre-trained generative model fine-tuned on ccRCC serum metabolomics
    whose representations drive prognostic and predictive stratification
  repo: https://github.com/ReviveMed/mzEmbed
- name: mzLearn
  role: Generates the 2,736 robust metabolite-feature MS1 matrix from raw mzML that
    feeds the fine-tuned VAE prior to stratification
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:4efdb70aa3810a1d63623458d6167afd44e609e75797646dad1e2d29990761e5
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
  - VAE
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kaplan-meier-survival-stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratify metabolomics cohorts by model-predicted risk or response and compare overall survival distributions with Kaplan-Meier curves and log-rank tests. Applied here to validate a fine-tuned VAE against IMDC risk criteria on held-out ccRCC serum samples.

## When to use

Use when patient-level metadata includes overall survival time/event and predicted risk groups (e.g. VAE embeddings or IMDC classes) and you must test whether groups separate by OS via KM curves and log-rank p-values.

## When NOT to use

- Input feature tables are not patient-level with survival time/event metadata — KM and log-rank tests are undefined without OS labels
- The cohort is too small to estimate KM curves with meaningful follow-up; the paper's held-out test set was n=143 ccRCC patients
- The question is feature-level metabolite importance rather than patient-level survival separation (use SHAP analysis instead)

## Inputs

- Fine-tuned VAE model checkpoint (best_finetune_model.pt) and saved config
- Test-cohort feature matrix X_Finetune_Test.csv with 2,736 mzLearn features
- Survival labels: overall survival time and event columns (y_Finetune_Test.csv)
- IMDC risk-class assignments per patient (binary or 3-class)
- Treatment-arm labels (ICI vs mTOR) for predictive-response analysis

## Outputs

- Kaplan-Meier survival curves per predicted risk group
- Log-rank p-values for VAE-defined vs IMDC-defined risk-group comparisons
- Hazard ratios for the prognostic stratification
- Predicted-response group assignments per treatment arm (ICI, mTOR)
- Results table (e.g. fig5_validation_results.csv) with p-values and stratification assignments

## How to apply

Score each held-out patient with the fine-tuned VAE encoder (from best_finetune_model.pt) and split the cohort into predicted risk groups; in parallel, use the clinical IMDC assignment for comparison. Run Kaplan-Meier estimation of overall survival per group and log-rank tests between groups, requiring the prognostic VAE to produce significantly distinct OS distributions for all predicted groups while IMDC only separates poor vs favorable. For treatment-response stratification (ICI vs mTOR arms), stratify by the predictive VAE's predicted response and compute KM curves with log-rank tests per arm; a useful classifier shows significant separation in the ICI arm (reported p=2.1e-3) but not mTOR (p=0.61). Write log-rank p-values, stratification assignments, and per-patient group labels to a results table such as fig5_validation_results.csv, and report hazard ratios alongside p-values to quantify stratification improvement.

## Related tools

- **mzEmbed** (Provides the fine-tuned VAE encoder whose latent predictions define the risk groups tested with KM/log-rank; test evaluation entry point returns metrics on held-out data) — https://github.com/ReviveMed/mzEmbed
- **VAE** (Supplies the pre-trained generative model fine-tuned on ccRCC serum metabolomics whose representations drive prognostic and predictive stratification) — https://github.com/ReviveMed/mzEmbed
- **mzLearn** (Generates the 2,736 robust metabolite-feature MS1 matrix from raw mzML that feeds the fine-tuned VAE prior to stratification)

## Examples

```
python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task cox
```

## Evaluation signals

- Predictive-model ICI arm log-rank p-value is significant (reported 2.1e-3) while mTOR arm is not (reported 0.61)
- Prognostic VAE produces significantly distinct OS distributions for all predicted risk groups, whereas IMDC only separates poor vs favorable
- Hazard ratios from the VAE-defined stratification improve relative to the IMDC gold standard
- Predicted-response group assignments cover all held-out patients (n=143) with no missing labels
- Results table contains log-rank p-values and stratification assignments for both prognostic and predictive arms

## Limitations

- Kaplan-Meier/log-rank comparison against IMDC was performed on a single held-out ccRCC test set (n=143); generalizability across multiple fine-tuning datasets is untested
- The ICI-response classification lacks additional validation for clinical application, so stratification p-values alone do not establish clinical utility
- Fine-tuned VAE predictions depend on the HILIC-positive-mode pre-trained model; other LC modes are not covered, limiting the feature space underlying the stratification
- Log-rank significance between the ICI arm (p=2.1e-3) and mTOR arm (p=0.61) reflects arm-specific predictive power rather than a universal classifier

## Evidence

- [discussion] The fine-tuned model was fine-tuned on baseline serum metabolomics data of ccRCC patients and matched or potentially outperformed the IMDC criteria for survival prediction: [evidence span withheld — anchor status: not_found]
- [readme] README documents the held-out test-evaluation entry point that returns survival metrics for the stratification workflow: [evidence span withheld — anchor status: external_doc]
- [readme] README specifies the cox/survival task option under which KM stratification is applied: [evidence span withheld — anchor status: not_found]
- [readme] README documents survival labels as required task-specific columns: [evidence span withheld — anchor status: not_found]
