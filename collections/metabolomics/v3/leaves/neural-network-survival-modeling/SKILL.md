---
name: neural-network-survival-modeling
description: Use when you have metabolomics data mapped to pathway-level PDS features
  plus patient survival times/events and want to fit or compare a neural-network-based
  survival model (Cox-nnet) against a Cox-PH baseline.
when_to_use_negative:
- Input is already pathway-level or has no paired survival time/event data — use Lilikoi's
  classification modules instead
- No Python environment available or python.path not configurable — Cox-nnet will
  not run without it
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
tools:
- name: Cox-nnet
  role: Deep-learning-based neural network survival model trained on PDS features
    and evaluated by C-index/log-rank
- name: Lilikoi v2.0
  role: R package providing lilikoi.prognosis with coxnnet=TRUE to run Cox-nnet and
    Cox-PH baselines
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Environment in which Lilikoi v2.0 and its prognosis module execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:9a4dc21e913bce21ea10dc08691895d3539233f5f818dda06c54eaee7ef60592
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# neural-network-survival-modeling

## Summary

Trains the Cox-nnet neural-network survival model — Lilikoi v2.0's deep-learning-based alternative to penalized Cox-PH — on pathway deregulation score (PDS) features derived from metabolomics data, and evaluates it with the concordance index and log-rank test. Grounded in the NCI breast tumor dataset where Cox-nnet reached a C-index of 0.66 vs 0.64 for Cox-PH.

## When to use

Use when you have metabolomics data mapped to pathway-level PDS features plus patient survival times/events and want to fit or compare a neural-network-based survival model (Cox-nnet) against a Cox-PH baseline.

## When NOT to use

- Input is already pathway-level or has no paired survival time/event data — use Lilikoi's classification modules instead
- No Python environment available or python.path not configurable — Cox-nnet will not run without it

## Inputs

- Metabolomics intensity matrix (e.g. plasma_breast_cancer.csv with 67 tumor samples, 536 metabolites)
- Pathway deregulation score (PDS) matrix from lilikoi.PDSfun
- Patient event indicators (event)
- Patient survival times (time)
- Selected significant pathways from lilikoi.featuresSelection

## Outputs

- Trained Cox-nnet survival model and Cox-PH baseline model
- Concordance index (C-index) per model
- Log-rank test P value for high- vs low-risk stratification
- Kaplan-Meier risk-group curves

## How to apply

In Lilikoi v2.0, first transform metabolite intensities into pathway deregulation score (PDS) features via lilikoi.MetaTOpathway and lilikoi.PDSfun, then pass the PDS matrix together with event indicators and survival times into lilikoi.prognosis with coxnnet=TRUE (gradient method). The same call fits the penalized Cox-PH baseline when coxnnet=FALSE; set alpha=0 (Ridge), nfold=5 cross-validation for lambda selection, and a feature-selection threshold for pathway inclusion. Evaluate both models with the concordance index and Kaplan-Meier log-rank test on high- vs low-risk groups; on the NCI breast tumor data (67 samples, 536 metabolites) the expected reference performance is Cox-nnet C-index 0.66 (log-rank P 0.02) vs Cox-PH C-index 0.64 (P 0.04). If a Python interpreter is not on the default path, supply it via python.path because Cox-nnet requires Python.

## Related tools

- **Cox-nnet** (Deep-learning-based neural network survival model trained on PDS features and evaluated by C-index/log-rank)
- **Lilikoi v2.0** (R package providing lilikoi.prognosis with coxnnet=TRUE to run Cox-nnet and Cox-PH baselines) — https://github.com/lanagarmire/lilikoi2
- **R** (Environment in which Lilikoi v2.0 and its prognosis module execute)

## Examples

```
lilikoi.prognosis(event, time, PDSmatrix, alpha=0, nfold=5, cvlambda=cvlambda, python.path=Sys.which("python"), coxnnet=TRUE, coxnnet_method="gradient")
```

## Evaluation signals

- Cox-nnet C-index should reach ~0.66 and log-rank P ~0.02 on the NCI breast tumor dataset; Cox-PH baseline ~0.64 / 0.04
- Cox-PH must be run with Ridge penalty (alpha=0) and 5-fold cross-validated lambda (cvlambda) as in the reference workflow
- Both models must be trained on the same PDS feature matrix with a first-quartile pathway impact threshold applied for feature inclusion
- Log-rank test on KM high/low-risk split should be significant (P < 0.05) for the trained Cox-nnet model

## Limitations

- Cox-nnet depends on a Python runtime that must be supplied via python.path; missing configuration causes failure
- Feature inclusion depends on a pathway impact threshold (first quartile), so low-impact pathways are excluded and results can shift with threshold choice
- Reference performance is dataset-specific (NCI breast tumor metabolomics) and may not generalize to other cohorts

## Evidence

- [results] Cox-nnet slightly outperforms Cox-PH on NCI data: [evidence span withheld — anchor status: not_found]
- [readme] Prognosis module signature from README: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract statement of the two survival models: "the most significant addition of prognosis prediction, implemented by Cox-proportional hazards model and the deep learning–based Cox-nnet model."
- [readme] PDS feature construction from README: "PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
