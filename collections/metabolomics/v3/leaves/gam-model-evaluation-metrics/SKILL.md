---
name: gam-model-evaluation-metrics
description: Use when a fit_gam spline has been fitted on RT ordered pairs and you
  need quantitative accuracy (mean absolute error, per-compound |error| ≤ 0.1 min)
  on held-out known metabolites before trusting downstream alignment scores.
when_to_use_negative:
- Datasets come from excessively disparate chromatographic techniques (e.g. RPLC vs
  HILIC) where elution order is not preserved — the RT mapping itself, and hence its
  error metrics, are not meaningful.
- Abundance or RT values have been normalized or transformed so that ranked abundance/RT
  order information is destroyed, invalidating the anchor-based evaluation.
- Evaluating fit_loess rather than GAM models — the scaled-t family rationale and
  mgcv backing described here apply to fit_gam specifically.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: metabCombiner
  role: R package providing the fit_gam function, anchor selection (selectAnchors),
    and plasma20/plasma30 example datasets used to train and evaluate the RT-mapping
    GAM
  repo: https://github.com/hhabra/metabCombiner
- name: fit_gam
  role: Fits the Generalized Additive Model (modified mgcv gam) with scaled-t family
    used for retention-time mapping whose predictions are evaluated here
  repo: https://github.com/hhabra/metabCombiner
- name: mgcv
  role: R package whose gam function is the underlying implementation for fit_gam;
    the scat (scaled t) family argument is set to damp outlier influence
- name: selectAnchors
  role: Selects retention-time ordered pairs from known compounds that form the training
    set whose prediction accuracy the evaluation measures
  repo: https://github.com/hhabra/metabCombiner
- name: R
  role: Runtime environment for the metabCombiner package and evaluation scripts
  repo: https://github.com/hhabra/metabCombiner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:7f4617c7771fd4c6bdfa541e7a1fcbdf4a853cd85f76b785da80bbadd4c457dc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hhabra/metabCombiner
  repo_url: https://github.com/hhabra/metabCombiner
  tools:
  - GAM
  - R
  - fit_gam
  - metabCombiner
  - mgcv
  - selectAnchors
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# gam-model-evaluation-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate the quality of a fit_gam (GAM) retention-time mapping model by computing mean absolute prediction error on held-out known-feature anchors, using the scaled-t family to resist outlier influence. This is the metric-based validation step behind metabCombiner's reported ~0.06 min RT error on plasma datasets.

## When to use

Use when a fit_gam spline has been fitted on RT ordered pairs and you need quantitative accuracy (mean absolute error, per-compound |error| ≤ 0.1 min) on held-out known metabolites before trusting downstream alignment scores.

## When NOT to use

- Datasets come from excessively disparate chromatographic techniques (e.g. RPLC vs HILIC) where elution order is not preserved — the RT mapping itself, and hence its error metrics, are not meaningful.
- Abundance or RT values have been normalized or transformed so that ranked abundance/RT order information is destroyed, invalidating the anchor-based evaluation.
- Evaluating fit_loess rather than GAM models — the scaled-t family rationale and mgcv backing described here apply to fit_gam specifically.

## Inputs

- Fitted GAM RT-mapping model (output of fit_gam on anchor ordered pairs)
- Held-out set of known shared compounds with measured RTs in both datasets (e.g. 68 known metabolites from plasma20/plasma30)
- Predicted RT values for the test compounds

## Outputs

- Mean absolute retention time prediction error (minutes)
- Count/fraction of test compounds predicted within 0.1 min of their measured RT
- Comparison of semisupervised vs unsupervised model performance

## How to apply

After fitting the RT-mapping GAM with fit_gam (mgcv-backed, scaled-t 'scat' family for heavy-tailed data), split known shared compounds (e.g. the 68 known metabolites in the plasma20/plasma30 pair) into training anchors and test compounds. Predict RTs for test compounds via the fitted model and compute the mean absolute prediction error and the count of test compounds predicted within 0.1 min. On the plasma pair this yields MAE ≈ 0.06 min with >50/68 test compounds within 0.1 min; semisupervised models show a slight advantage over unsupervised ones. Judge success against these reported figures; unstable or inflated MAE indicates outlier anchor contamination or unsuitable family choice.

## Related tools

- **metabCombiner** (R package providing the fit_gam function, anchor selection (selectAnchors), and plasma20/plasma30 example datasets used to train and evaluate the RT-mapping GAM) — https://github.com/hhabra/metabCombiner
- **fit_gam** (Fits the Generalized Additive Model (modified mgcv gam) with scaled-t family used for retention-time mapping whose predictions are evaluated here) — https://github.com/hhabra/metabCombiner
- **mgcv** (R package whose gam function is the underlying implementation for fit_gam; the scat (scaled t) family argument is set to damp outlier influence)
- **selectAnchors** (Selects retention-time ordered pairs from known compounds that form the training set whose prediction accuracy the evaluation measures) — https://github.com/hhabra/metabCombiner
- **R** (Runtime environment for the metabCombiner package and evaluation scripts) — https://github.com/hhabra/metabCombiner

## Examples

```
library(metabCombiner); anchors <- selectAnchors(pairedData); model <- fit_gam(anchors, family = "scat"); pred <- predictRT(model, testCompounds); mae <- mean(abs(pred - testRT))
```

## Evaluation signals

- Mean absolute RT prediction error on held-out known compounds should be roughly 0.06 min on the plasma20/plasma30 pair
- More than 50 of the 68 known test compounds should be predicted within 0.1 min of their measured RT
- Semisupervised model MAE should be slightly lower than or comparable to the unsupervised model MAE
- MAE should be stable across repeat fits — inconsistent or inflated errors suggest outlier anchors were not excluded from the training set

## Limitations

- Achieving low error requires that elution order of metabolites be mostly preserved; HILIC-acquired datasets are harder to align than RPLC ones.
- The scaled-t ('scat') family is chosen specifically to limit the influence of outlier anchor points; using a Gaussian family can inflate MAE.
- Reported metrics are demonstrated on a single plasma pair; performance on other chromatographic regimes or laboratories is not established by these numbers.
- Input datasets must be in the same ionization mode and biologically similar samples with strong metabolomic overlap for the anchors to be representative.

## Evidence

- [abstract] Mean absolute RT prediction error ≈ 0.06 min on plasma pair: "achieving a mean absolute retention time prediction error of roughly 0.06 min"
- [abstract] Weighted per-compound matching accuracy of approximately 90% on the plasma data set pair: "a weighted per-compound matching accuracy of approximately 90%"
- [discussion] Scaled-t family in fit_gam suppresses outlier influence: "In particular, setting the default “family” argument to scat (scaled t-family for heavy-tailed data) helps to eliminate the influence of outlier points"
- [intro] metabCombiner is an R package for aligning disparately-acquired LC-MS metabolomics data: "This is an R package for aligning a pair of disparately-acquired untargeted LC-MS metabolomics."
- [intro] fit_gam is a modified mgcv gam; the scat family argument is set for heavy-tailed data: "This guide will mostly cover `fit_gam`, a modified form of the `gam` function implemented in the *mgcv* R package."
