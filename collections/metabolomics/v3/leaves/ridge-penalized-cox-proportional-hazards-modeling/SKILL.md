---
name: ridge-penalized-cox-proportional-hazards-modeling
description: Use when metabolomics samples have patient survival times and event indicators
  and the goal is prognosis prediction from pathway-level features (e.g., 67 NCI breast
  tumor samples × 536 metabolites reduced to PDS features).
when_to_use_negative:
- The analysis goal is disease classification (case/control labeling) rather than
  survival prediction — use Lilikoi's machine-learning/DL classification module instead
  (lilikoi.machine_learning).
- Input features are raw metabolite abundances that have not yet been transformed
  to pathway deregulation scores; run MetaTOpathway/PDSfun first.
- No survival times or event indicators are available for the samples — the Cox framework
  is inapplicable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: Lilikoi v2.0
  role: R package whose lilikoi.prognosis function fits the penalized Cox-PH model
    on PDS features
  repo: github.com/lanagarmire/lilikoi2
- name: R
  role: programming environment in which Lilikoi v2.0 and the Cox-PH prognosis modeling
    run
- name: Cox-nnet
  role: neural-network-based survival model trained on the same PDS features as benchmark
    comparison
  repo: github.com/lanagarmire/lilikoi2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:e9f171bea118e8e3855a5142174334ccab3a4c8ea85e2c9bba08b2eaeb635bd8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cox-nnet
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ridge-penalized-cox-proportional-hazards-modeling

## Summary

Fits a penalized Cox proportional hazards survival model on pathway deregulation score (PDS) features derived from metabolomics data, as implemented in Lilikoi v2.0's prognosis module, and evaluates risk stratification by concordance index and log-rank test with Kaplan-Meier curves.

## When to use

Use when metabolomics samples have patient survival times and event indicators and the goal is prognosis prediction from pathway-level features (e.g., 67 NCI breast tumor samples × 536 metabolites reduced to PDS features).

## When NOT to use

- The analysis goal is disease classification (case/control labeling) rather than survival prediction — use Lilikoi's machine-learning/DL classification module instead (lilikoi.machine_learning).
- Input features are raw metabolite abundances that have not yet been transformed to pathway deregulation scores; run MetaTOpathway/PDSfun first.
- No survival times or event indicators are available for the samples — the Cox framework is inapplicable.

## Inputs

- metabolomics feature matrix (e.g., NCI breast tumor dataset: 67 tumor samples, 536 metabolites)
- patient survival times (continuous)
- event indicators (binary censoring flags)
- pathway deregulation score (PDS) feature matrix from Lilikoi PDSfun
- cross-validated lambda (cvlambda) from Cox model tuning

## Outputs

- fitted ridge-penalized Cox-PH model on PDS features
- concordance index (C-index) of survival prediction
- log-rank test P value for high- vs low-risk group separation
- Kaplan-Meier survival curves for risk groups
- per-sample risk scores

## How to apply

First transform the metabolite matrix (e.g., 67 tumor samples, 536 metabolites) into pathway deregulation score (PDS) features via Lilikoi's MetaTOpathway and PDSfun functions, then apply a first-quartile pathway impact threshold for feature inclusion. Fit the penalized Cox-PH model on the PDS features with Ridge regularization, setting alpha=0 (ridge penalty) and selecting the penalty via cross-validated lambda (cvlambda) over 5 folds with the 'quantile' method for risk-group splitting; in Lilikoi this is done with coxnnet=FALSE to keep the neural-network branch off. Evaluate by concordance index and a log-rank test comparing high- vs low-risk Kaplan-Meier curves; on the NCI breast tumor data this reproduces C-index 0.64 with log-rank P = 0.04. For benchmarking, train Cox-nnet on the same PDS features and compare (Cox-nnet achieved C-index 0.66, log-rank P = 0.02).

## Related tools

- **Lilikoi v2.0** (R package whose lilikoi.prognosis function fits the penalized Cox-PH model on PDS features) — github.com/lanagarmire/lilikoi2
- **R** (programming environment in which Lilikoi v2.0 and the Cox-PH prognosis modeling run)
- **Cox-nnet** (neural-network-based survival model trained on the same PDS features as benchmark comparison) — github.com/lanagarmire/lilikoi2

## Examples

```
lilikoi.prognosis(event, time, exprdata, percent=percent, alpha=0, nfold=5, method="quantile", cvlambda=cvlambda, python.path=NULL, coxnnet=FALSE, coxnnet_method="gradient")
```

## Evaluation signals

- Reproduces the reported metrics on the NCI breast tumor dataset: Cox-PH C-index 0.64 with log-rank P = 0.04
- Benchmark comparison holds: Cox-nnet achieves C-index 0.66 with log-rank P = 0.02 on the same PDS features
- Kaplan-Meier curves for high- vs low-risk groups separate significantly under the log-rank test
- Ridge penalty is confirmed by alpha=0 in the lilikoi.prognosis call and lambda selected by 5-fold cross-validation (cvlambda)
- Only pathways passing the first-quartile pathway impact threshold enter the model

## Limitations

- Performance was demonstrated on a small cohort (67 tumor samples), so metrics like C-index 0.64 may vary on other cohorts
- Cox-PH was slightly outperformed by the deep-learning Cox-nnet model (C-index 0.66 vs 0.64) on this dataset
- The Cox-nnet branch of lilikoi.prognosis requires a Python interpreter (python.path) and defaults to gradient-based optimization, adding a dependency if benchmarking

## Evidence

- [abstract] prognosis prediction metrics: "the most significant addition of prognosis prediction, implemented by Cox-proportional hazards model and the deep learning–based Cox-nnet model."
- [readme] workflow parameterization: [evidence span withheld — anchor status: external_doc]
- [readme] pathway feature construction: "PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
- [intro] rationale for survival modeling in metabolomics: "metabolomics data have the potential to be prognosis markers [10]; however, at present it is rare for a metabolomics data analysis workflow to be available for prognosis modeling and prediction."
- [results] reported performance: [evidence span withheld — anchor status: not_found]
