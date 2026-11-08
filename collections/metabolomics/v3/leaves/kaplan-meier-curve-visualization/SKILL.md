---
name: kaplan-meier-curve-visualization
description: Use when metabolomics samples have survival times and event indicators
  and you need to visually validate risk-group stratification from a fitted Cox-PH
  or Cox-nnet model on pathway deregulation score (PDS) features.
when_to_use_negative:
- The analysis goal is disease classification rather than survival prognosis — use
  Lilikoi's classification module (lilikoi.machine_learning) instead.
- Input is a raw metabolite matrix that has not yet been transformed to pathway-level
  PDS features via lilikoi.MetaTOpathway and lilikoi.PDSfun.
- Samples lack survival times and event indicators, so no risk-group stratification
  over time can be evaluated.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3382
tools:
- name: Lilikoi v2.0
  role: R package whose prognosis module fits the Cox-PH/Cox-nnet models and supports
    survival evaluation with log-rank tests and Kaplan-Meier curves on PDS features
  repo: https://github.com/lanagarmire/lilikoi2
- name: Cox-nnet
  role: Deep learning survival model whose predicted risk scores define the high-
    vs low-risk strata visualized in the KM curves
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which Lilikoi v2.0 and the KM/log-rank evaluation
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:3e5eec03d769abcbb5c30bea95dc75ff41e3502d06b14d34e1be2cf90627abef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Cox-nnet
  - Lilikoi v2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# kaplan-meier-curve-visualization

## Summary

Visualize survival differences between high- and low-risk patient groups stratified by a Cox-based prognostic model (Cox-PH or Cox-nnet) using Kaplan-Meier curves with log-rank tests, as implemented in the Lilikoi v2.0 prognosis module for pathway-level metabolomics data.

## When to use

Use when metabolomics samples have survival times and event indicators and you need to visually validate risk-group stratification from a fitted Cox-PH or Cox-nnet model on pathway deregulation score (PDS) features.

## When NOT to use

- The analysis goal is disease classification rather than survival prognosis — use Lilikoi's classification module (lilikoi.machine_learning) instead.
- Input is a raw metabolite matrix that has not yet been transformed to pathway-level PDS features via lilikoi.MetaTOpathway and lilikoi.PDSfun.
- Samples lack survival times and event indicators, so no risk-group stratification over time can be evaluated.

## Inputs

- PDS (pathway deregulation score) feature matrix derived from metabolite data (e.g., 67 NCI breast tumor samples across pathway features)
- Patient survival times (time)
- Event indicators (event)
- Fitted prognostic model risk scores from lilikoi.prognosis (Cox-PH or Cox-nnet)

## Outputs

- Kaplan-Meier survival curves for high- vs low-risk groups
- Log-rank test P value between risk groups
- Concordance index (C-index) of the prognostic model

## How to apply

After fitting a penalized Cox-PH model (Ridge regularization, alpha=0, 5-fold cross-validated lambda) or a Cox-nnet neural network survival model on pathway-level PDS features, split samples into high- vs low-risk groups using a quantile-based risk-score cutoff (method='quantile', percent=percent). Draw Kaplan-Meier survival curves for the two risk groups over patient survival times and compute the log-rank test P value comparing them; also report the concordance index (C-index) for model discrimination. Correct application is judged by whether the KM curves separate visually and the log-rank P value reaches significance (e.g., on the NCI breast tumor dataset, Cox-PH gave C-index 0.64 with log-rank P=0.04 and Cox-nnet gave C-index 0.66 with log-rank P=0.02).

## Related tools

- **Lilikoi v2.0** (R package whose prognosis module fits the Cox-PH/Cox-nnet models and supports survival evaluation with log-rank tests and Kaplan-Meier curves on PDS features) — https://github.com/lanagarmire/lilikoi2
- **Cox-nnet** (Deep learning survival model whose predicted risk scores define the high- vs low-risk strata visualized in the KM curves) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which Lilikoi v2.0 and the KM/log-rank evaluation are executed)

## Examples

```
lilikoi.prognosis(event, time, exprdata, percent=percent, alpha=0, nfold=5, method='quantile', cvlambda=cvlambda, coxnnet=TRUE, coxnnet_method='gradient')
```

## Evaluation signals

- KM curves show visible separation between high- and low-risk groups with a log-rank P value reported (e.g., P=0.04 for Cox-PH, P=0.02 for Cox-nnet on the NCI breast tumor data)
- Concordance index matches expected performance ranges (C-index ~0.64 for Cox-PH, ~0.66 for Cox-nnet on the NCI breast tumor dataset)
- Risk-group split follows the configured quantile cutoff (method='quantile', percent=percent) rather than arbitrary assignment
- Curves are computed on the same PDS feature matrix used to fit the Cox model, ensuring train/evaluate consistency

## Limitations

- Performance is dataset-dependent; the reported metrics apply to the NCI breast tumor metabolomics dataset (67 tumor samples, 536 metabolites mapped to pathways) and may not transfer to other cohorts.
- The evaluation depends on upstream pathway mapping quality — metabolite-to-pathway assignment errors in MetaTOpathway/PDSfun propagate into the risk scores and KM curves.
- KM curves alone show group separation but not continuous discrimination; they should be interpreted alongside the C-index.

## Evidence

- [abstract] Cox-PH and Cox-nnet prognosis with KM evaluation: "the most significant addition of prognosis prediction, implemented by Cox-proportional hazards model and the deep learning–based Cox-nnet model."
- [intro] Rationale for prognosis modeling in metabolomics: "metabolomics data have the potential to be prognosis markers [10]; however, at present it is rare for a metabolomics data analysis workflow to be available for prognosis modeling and prediction."
- [readme] lilikoi.prognosis invocation for risk scoring: [evidence span withheld — anchor status: external_doc]
- [readme] Prognosis module operates on PDS-transformed pathway features: "PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
