---
name: greenwood-confidence-interval-computation
description: Use when comparing survival or cumulative-event curves between risk groups
  (e.g., MetaboRS tertiles) and you need CI bands to judge whether group curves are
  statistically separated.
when_to_use_negative:
- You have only the fine-tuned model's probability column and no follow-up time or
  event labels — CI computation requires duration/event data
- Groups are not defined by MetaboRS tertiles or a comparable risk stratification
  with pre-specified cutoffs
- Input is raw metabolite abundance that still needs imputation and prediction via
  the metabolome-transformer pipeline before stratification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: lifelines
  role: KaplanMeierFitter estimates per-tertile cumulative event curves and the Greenwood-variance-based
    95% confidence intervals
- name: matplotlib
  role: Plots the three Kaplan-Meier curves with CI bands and saves kaplan_meier_curves.png
- name: pandas
  role: Holds tertile assignments and computes the per-disease high-vs-low incidence
    risk ratios
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-66163-3
    title: 'MetaboLM: a metabolomic language model for multi-disease early prediction
      and risk stratification'
  source_content_hash: sha256:de1fc95f26452e5671664f8f420e74dc95962f428c36bdea61e793674316fe7c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Qiu-Shizheng/MetaboLM
  tools:
  - lifelines
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# greenwood-confidence-interval-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes 95% confidence intervals around Kaplan-Meier cumulative event curves using the Greenwood variance formula, applied here to MetaboRS-derived risk tertiles (high >0.66, low <0.33) for 16 chronic diseases in the MetaboLM pipeline.

## When to use

Use when comparing survival or cumulative-event curves between risk groups (e.g., MetaboRS tertiles) and you need CI bands to judge whether group curves are statistically separated.

## When NOT to use

- You have only the fine-tuned model's probability column and no follow-up time or event labels — CI computation requires duration/event data
- Groups are not defined by MetaboRS tertiles or a comparable risk stratification with pre-specified cutoffs
- Input is raw metabolite abundance that still needs imputation and prediction via the metabolome-transformer pipeline before stratification

## Inputs

- MetaboRS tertile assignment per participant (high >0.66, medium, low <0.33 probability cutoffs)
- Per-participant follow-up durations and binary disease-event indicators for the 16 chronic diseases
- Fine-tuned MetaboLM prediction probabilities per participant per disease

## Outputs

- Kaplan-Meier cumulative event curves for high/medium/low MetaboRS tertiles with Greenwood-based 95% CI bands (kaplan_meier_curves.png)
- Tertile assignments and high-vs-low risk ratios written to risk_stratification_results.csv

## How to apply

After stratifying participants by MetaboRS prediction probability into high (>0.66), low (<0.33), and medium tertiles, fit a KaplanMeierFitter (lifelines) per risk group on follow-up duration and event-occurrence columns to obtain cumulative disease-event curves. Compute the 95% confidence band for each curve via the Greenwood variance formula, which lifelines exposes alongside the survival function. Plot the three tertile curves with their CI bands using matplotlib and save as kaplan_meier_curves.png. The validation criterion is that the high-risk group's cumulative event curve lies above the low-risk group's curve with non-overlapping Greenwood-based 95% CIs, alongside the >=726-fold high-vs-low risk ratio check written to risk_stratification_results.csv.

## Related tools

- **lifelines** (KaplanMeierFitter estimates per-tertile cumulative event curves and the Greenwood-variance-based 95% confidence intervals)
- **matplotlib** (Plots the three Kaplan-Meier curves with CI bands and saves kaplan_meier_curves.png)
- **pandas** (Holds tertile assignments and computes the per-disease high-vs-low incidence risk ratios)

## Examples

```
from lifelines import KaplanMeierFitter; kmf = KaplanMeierFitter(); kmf.fit(df['duration'], df['event']); ci = kmf.confidence_interval_survival_function_
```

## Evaluation signals

- High-risk group's Kaplan-Meier cumulative event curve lies above the low-risk group's curve for every disease examined
- Greenwood-based 95% CI bands of the high- and low-risk groups do not overlap over the follow-up window
- High-vs-low risk ratio is at least 726-fold for the reported diseases
- risk_stratification_results.csv contains one row per participant-disease with valid tertile labels (high/medium/low) and finite risk ratios
- All three tertile curves are present in kaplan_meier_curves.png with CI bands rendering without gaps

## Limitations

- The MetaboLM cohort is predominantly of European descent, healthier, and of higher socioeconomic status, which may limit generalizability of the tertile survival comparisons
- NMR-based metabolomics may miss metabolites detectable by other analytical methods, affecting upstream prediction probabilities that drive tertile assignment
- Fixed cutoffs (>0.66 / <0.33) assume model probabilities are calibrated; uncalibrated fine-tuned classifiers could distort tertile membership

## Evidence

- [other] Estimate Kaplan-Meier cumulative event curves for each risk tertile using KaplanMeierFitter in lifelines, with 95% confidence intervals computed via the Greenwood variance formula.: [evidence span withheld — anchor status: not_found]
- [other] CI non-overlap validation criterion: [evidence span withheld — anchor status: not_found]
- [other] Tertile cutoffs from the source card: [evidence span withheld — anchor status: not_found]
- [intro] Prediction probabilities feed the risk score: "By leveraging prediction probabilities derived from the metabolomic profile, we developed a metabolomic risk score (MetaboRS) capable of stratifying individuals"
- [readme] Classifier probability output format: [evidence span withheld — anchor status: external_doc]
