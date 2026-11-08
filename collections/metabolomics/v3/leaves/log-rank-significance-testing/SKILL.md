---
name: log-rank-significance-testing
description: Use when comparing overall survival distributions between predicted risk
  groups (e.g. VAE-derived or IMDC-defined) on held-out patient cohorts and needing
  a significance p-value.
when_to_use_negative:
- Input is already a feature importance list or SHAP ranking — the log-rank test applies
  to survival outcomes, not feature scores.
- Only a single survival curve (no group comparison) exists; log-rank requires ≥2
  groups to test.
- Data lacks time-to-event outcome (OS time/event or censoring); the test is meaningless
  on cross-sectional labels.
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0203
tools:
- name: mzLearn
  role: Generates the MS1 feature matrix whose fine-tuned model outputs (risk-group
    assignments) are the groups whose survival curves are compared via log-rank.
  repo: https://github.com/ReviveMed/mzEmbed
- name: VAE
  role: Pre-trained variational autoencoder fine-tuned on renal cell carcinoma data
    to produce the patient stratification that the log-rank test validates against
    IMDC criteria.
  repo: https://github.com/ReviveMed/mzEmbed
- name: SHAP
  role: Feature-importance analysis on the fine-tuned models, whose Top-20 metabolite
    lists are a companion validation output but not directly consumed by the log-rank
    test itself.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:910905997e7a1ed42bb01c5c8d5d4c50f2caaa8f9b91c834362c47ce0c0b3b61
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
  - SHAP
  - VAE
  - mzLearn
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# log-rank-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A survival-analysis significance test that compares Kaplan-Meier curves between risk groups by computing a log-rank p-value. In this paper it is used to validate that VAE-derived stratifications of ccRCC patients match or outperform IMDC clinical criteria on held-out test data.

## When to use

Use when comparing overall survival distributions between predicted risk groups (e.g. VAE-derived or IMDC-defined) on held-out patient cohorts and needing a significance p-value.

## When NOT to use

- Input is already a feature importance list or SHAP ranking — the log-rank test applies to survival outcomes, not feature scores.
- Only a single survival curve (no group comparison) exists; log-rank requires ≥2 groups to test.
- Data lacks time-to-event outcome (OS time/event or censoring); the test is meaningless on cross-sectional labels.

## Inputs

- Held-out test cohort with per-patient overall survival times, censoring events, and treatment-arm labels (e.g. ICI vs mTOR)
- Per-patient risk-group assignment from a fine-tuned prognostic or predictive VAE model
- Per-patient risk-group assignment from the IMDC clinical criteria baseline

## Outputs

- Kaplan-Meier survival curves per risk group
- Log-rank p-values per stratification and treatment arm
- Hazard ratios comparing model-derived vs IMDC risk groups
- Validation results table (e.g. fig5_validation_results.csv)

## How to apply

Score each test-cohort patient (e.g. the n=143 ccRCC test set) with a prognostic or treatment-predictive model and split into risk groups; then run Kaplan-Meier estimation per group and a log-rank test to quantify whether the survival curves differ significantly. Compare the resulting p-values and hazard ratios against a clinical baseline (IMDC risk grouping) to judge whether the model-derived stratification improves or matches the standard. Accept the stratification as reproduced/validated when the log-rank p-value is significant for the arm of interest (the ICI-response model reported p = 2.1e-3) while the negative-control arm (mTOR, reported p = 0.61) is not, confirming treatment-specific rather than general predictive signal.

## Related tools

- **mzLearn** (Generates the MS1 feature matrix whose fine-tuned model outputs (risk-group assignments) are the groups whose survival curves are compared via log-rank.) — https://github.com/ReviveMed/mzEmbed
- **VAE** (Pre-trained variational autoencoder fine-tuned on renal cell carcinoma data to produce the patient stratification that the log-rank test validates against IMDC criteria.) — https://github.com/ReviveMed/mzEmbed
- **SHAP** (Feature-importance analysis on the fine-tuned models, whose Top-20 metabolite lists are a companion validation output but not directly consumed by the log-rank test itself.)

## Examples

```
python -m mz_embed.finetune.best_finetune_model_test_eval --model_dir /path/to/out_finetune --data_location /path/to/finetune_csvs --task cox
```

## Evaluation signals

- Log-rank p-value for the ICI-response model arm should be significant (paper-reported benchmark ≈ 2.1e-3).
- Log-rank p-value for the mTOR negative-control arm should be non-significant (paper-reported ≈ 0.61), confirming treatment-specific signal.
- Prognostic VAE should show significance for all predicted risk groups and equal-or-better hazard ratios compared with IMDC, which is only significant for poor vs favorable.
- Validation results must be written to the expected output table (fig5_validation_results.csv) with stratification assignments present.
- Both SHAP Top-20 lists (if run alongside) should share 3 of 20 features including reported lipid and carnitine members.

## Limitations

- The ICI-response classification lacks additional validation for clinical application, so log-rank significance alone is insufficient evidence for treatment-decision use.
- Generalizability of fine-tuned models across multiple fine-tuning datasets is untested; log-rank results may not transfer beyond this cohort.
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, which can propagate into risk scores feeding the survival test.

## Evidence

- [discussion] VAE-derived stratification compared to IMDC criteria: [evidence span withheld — anchor status: not_found]
