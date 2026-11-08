---
name: machine-learning-performance-aggregation
description: Use when evaluating a multi-model, multi-class prediction tool (e.g.
  one model per CYP450 enzyme) and you need per-class cross-validation AUROC plus
  an aggregate average to compare against reported benchmarks (0.83-0.92 for CypReact
  vs 0.75 ADMET Predictor and 0.53 SMARTCyp).
when_to_use_negative:
- The task is reactant prediction on new query molecules with already-trained CypReact
  models, not performance evaluation of the models themselves
- Your evaluation metric is site-of-metabolism ranking (e.g. SMARTCyp-style top-N
  accuracy) rather than binary reactant/non-reactant AUROC
- The dataset is not a set of plausible reactant molecules — non-reactant or external
  sets require the hold-out evaluation protocol, not cross-validation aggregation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CypReact
  role: Primary tool whose learned per-enzyme models are run in cross-validation mode
    to produce the per-enzyme AUROC values being aggregated
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: ADMET Predictor
  role: Baseline reactant-prediction tool for comparison, reported at average AUROC
    0.75 versus CypReact's 0.83-0.92
- name: SMARTCyp
  role: Baseline reactant-predicting extension of SMARTCyp used for comparison, reported
    at average AUROC 0.53
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:f42ee2eff787e6d720d4f629d8bc4f42d3ca6d1273f0a4bfdb330363b5767043
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# machine-learning-performance-aggregation

## Summary

Reproduce and aggregate cross-validated classifier performance across multiple per-class models, following CypReact's protocol of computing per-enzyme AUROC values for all nine human CYP450 enzymes (CYP1A2 through CYP3A4) on the 1632-molecule plausible-reactant dataset and reporting both per-enzyme and average AUROC.

## When to use

Use when evaluating a multi-model, multi-class prediction tool (e.g. one model per CYP450 enzyme) and you need per-class cross-validation AUROC plus an aggregate average to compare against reported benchmarks (0.83-0.92 for CypReact vs 0.75 ADMET Predictor and 0.53 SMARTCyp).

## When NOT to use

- The task is reactant prediction on new query molecules with already-trained CypReact models, not performance evaluation of the models themselves
- Your evaluation metric is site-of-metabolism ranking (e.g. SMARTCyp-style top-N accuracy) rather than binary reactant/non-reactant AUROC
- The dataset is not a set of plausible reactant molecules — non-reactant or external sets require the hold-out evaluation protocol, not cross-validation aggregation

## Inputs

- 1632-molecule dataset of plausible reactant molecules for human CYP450 enzymes
- Learned per-enzyme CypReact models for the nine CYP450 isoforms (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4)

## Outputs

- Per-enzyme cross-validation AUROC values for each of the nine CYP450 enzymes
- Average AUROC across the nine enzymes
- Results table confirming whether values fall within the reported 0.83-0.92 range

## How to apply

Obtain the trained per-enzyme models and the 1632-molecule reactant dataset from the CypReact Bitbucket repository. Run cross-validation separately for each of the nine target enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4), computing the area under the receiver operating characteristic curve (AUROC) from the cross-validated reactant predictions for each enzyme. Aggregate the nine per-enzyme AUROC values into an average and assemble a results table with both per-enzyme and average values. Benchmark against the paper's reported range: CypReact AUROC 0.83-0.92, with ADMET Predictor at average AUROC 0.75 and SMARTCyp (reactant-predicting extension) at 0.53 as comparison baselines. Optionally repeat the aggregation on a previously unseen test set, where CypReact performed even better while still surpassing SMARTCyp and ADMET Predictor.

## Related tools

- **CypReact** (Primary tool whose learned per-enzyme models are run in cross-validation mode to produce the per-enzyme AUROC values being aggregated) — https://bitbucket.org/Leon_Ti/cypreact
- **ADMET Predictor** (Baseline reactant-prediction tool for comparison, reported at average AUROC 0.75 versus CypReact's 0.83-0.92)
- **SMARTCyp** (Baseline reactant-predicting extension of SMARTCyp used for comparison, reported at average AUROC 0.53)

## Evaluation signals

- Each of the nine per-enzyme cross-validation AUROC values falls within the reported 0.83-0.92 range
- The average AUROC across the nine CYP450 enzymes is consistent with the paper's reported average for CypReact
- Benchmark baselines reproduce in the expected order: CypReact > ADMET Predictor (average AUROC 0.75) > SMARTCyp (average AUROC 0.53)
- All nine enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) are represented in the results table with no missing per-enzyme values
- On a previously unseen molecule set, aggregated AUROC is at least as high as the cross-validation values, matching the paper's finding that performance improved on unseen data

## Limitations

- Cross-validation AUROC of 0.83-0.92 applies specifically to the 1632-molecule plausible-reactant dataset; performance on other distributions is not established by this protocol
- AUROC aggregates ranking quality only and does not directly reflect per-enzyme sensitivity/specificity at an operating threshold
- Comparison baselines (ADMET Predictor 0.75, SMARTCyp 0.53) are average AUROCs and may be computed on different cross-validation splits

## Evidence
