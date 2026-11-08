---
name: cross-validation-auroc-computation
description: 'Use when reproducing or benchmarking CypReact''s per-enzyme reactant
  prediction performance: you have the CypReact package, the 1632-molecule reactant
  dataset, and need cross-validation AUROC per CYP450 enzyme.'
when_to_use_negative:
- The goal is benchmarking ADMET Predictor or SMARTCyp themselves — this skill evaluates
  CypReact's cross-validation AUROC, not the baselines' pipelines.
- Evaluating on the previously unseen external molecule set — the paper reports CypReact
  'did even better' there, so the 0.83-0.92 cross-validation range would be the wrong
  validation target.
- The input is site-of-metabolism prediction output rather than a binary reactant/non-reactant
  classification per CYP450 enzyme.
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: CypReact
  role: 'Primary tool: provides the per-enzyme reactant-prediction models run in cross-validation
    mode to generate the AUROC scores'
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: ADMET Predictor
  role: Comparison baseline reactant-prediction tool (average AUROC 0.75) used to
    judge CypReact's cross-validation performance
- name: SMARTCyp
  role: Comparison baseline reactant-predicting extension (average AUROC 0.53) used
    to judge CypReact's cross-validation performance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:950698e34fd6d4ec5d8b101d20b73e22778282cabfb6293cea036468c123621f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-validation-auroc-computation

## Summary

Computes cross-validated AUROC (area under the receiver operating characteristic curve) values for the CypReact reactant-prediction models across nine human CYP450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4), reproducing the paper's reported 0.83-0.92 range on the 1632-molecule 'plausible reactant' dataset.

## When to use

Use when reproducing or benchmarking CypReact's per-enzyme reactant prediction performance: you have the CypReact package, the 1632-molecule reactant dataset, and need cross-validation AUROC per CYP450 enzyme.

## When NOT to use

- The goal is benchmarking ADMET Predictor or SMARTCyp themselves — this skill evaluates CypReact's cross-validation AUROC, not the baselines' pipelines.
- Evaluating on the previously unseen external molecule set — the paper reports CypReact 'did even better' there, so the 0.83-0.92 cross-validation range would be the wrong validation target.
- The input is site-of-metabolism prediction output rather than a binary reactant/non-reactant classification per CYP450 enzyme.

## Inputs

- CypReact package with learned per-enzyme models (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4)
- 1632-molecule dataset of 'plausible' reactant molecules with per-enzyme reactant labels

## Outputs

- Per-enzyme cross-validation AUROC values for each of the nine CYP450 enzymes
- Average AUROC across the nine enzymes (expected within 0.83-0.92)
- Results table with per-enzyme and average AUROC values

## How to apply

Obtain CypReact from its public Bitbucket repository (https://bitbucket.org/Leon_Ti/cypreact), which ships the learned per-enzyme models and the 1632-molecule reactant dataset. Run CypReact in cross-validation mode separately for each of the nine target enzymes: CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, and CYP3A4. For each enzyme, compute the area under the receiver operating characteristic curve (AUROC) from the cross-validated reactant predictions, then aggregate the nine per-enzyme AUROC values into an average. Assemble a results table with per-enzyme and average AUROC values and confirm they fall within the reported cross-validation range of 0.83-0.92. As a sanity benchmark, CypReact should outperform ADMET Predictor (average AUROC 0.75) and the reactant-predicting extension of SMARTCyp (average AUROC 0.53) under the same evaluation.

## Related tools

- **CypReact** (Primary tool: provides the per-enzyme reactant-prediction models run in cross-validation mode to generate the AUROC scores) — https://bitbucket.org/Leon_Ti/cypreact
- **ADMET Predictor** (Comparison baseline reactant-prediction tool (average AUROC 0.75) used to judge CypReact's cross-validation performance)
- **SMARTCyp** (Comparison baseline reactant-predicting extension (average AUROC 0.53) used to judge CypReact's cross-validation performance)

## Evaluation signals

- Per-enzyme AUROC values for all nine CYP450 enzymes fall within the reported 0.83-0.92 range.
- The average AUROC across the nine enzymes falls within 0.83-0.92.
- The evaluated dataset contains 1632 molecules, matching the paper's 'plausible reactant' set.
- CypReact's average AUROC exceeds ADMET Predictor's 0.75 and SMARTCyp's 0.53 average AUROCs.
- All nine target enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) are represented in the results table.

## Limitations

- The 0.83-0.92 range applies only to cross-validation on the 1632-molecule dataset; on a previously unseen set of molecules CypReact performed even better, so the range is not a universal expectation.
- AUROC comparisons to ADMET Predictor and SMARTCyp depend on those tools' reported average AUROCs (0.75 and 0.53) and may not hold under different datasets or evaluation protocols.

## Evidence

- [abstract] reported cross-validation AUROC range: "Tests of CypReact, conducted over a data set of 1632 molecules (each considered a "plausible" reactant) show that it is very effective, with a (cross-validation) AUROC (area under the receiver"
- [abstract] baseline comparison AUROCs: "CypReact performs significantly better than other reactant prediction tools such as ADMET Predictor and (a reactant-predicting extension of) SMARTCyp, whose average AUROCs are 0.75 and 0.53,"
- [abstract] tool availability: "CypReact is available at https://bitbucket.org/Leon_Ti/cypreact ."
