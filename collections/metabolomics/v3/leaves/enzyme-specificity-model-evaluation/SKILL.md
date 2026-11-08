---
name: enzyme-specificity-model-evaluation
description: Use when classifying whether query molecules react with specific human
  CYP450 isoforms and you need per-enzyme AUROC evaluation, cross-validation, or benchmarking
  against ADMET Predictor/SMARTCyp baselines.
when_to_use_negative:
- The goal is metabolite structure or exact site-of-metabolism (regioselectivity)
  prediction rather than the first-task reactant/non-reactant classification that
  CypReact performs
- The query molecules are not plausible CYP450 reactants (e.g. outside drug-like chemical
  space covered by the 1632-molecule training/test dataset), where the learned models'
  applicability is unverified
- You only have prediction scores with no ground-truth reactant labels, making AUROC
  computation and cross-validation impossible
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: CypReact
  role: Primary reactant-prediction tool whose per-enzyme learned models are run in
    cross-validation mode to produce AUROC-scored reactant predictions for the nine
    CYP450 enzymes
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: ADMET Predictor
  role: Baseline reactant-prediction tool used for comparison; expected average AUROC
    0.75 versus CypReact's 0.83-0.92
- name: SMARTCyp
  role: Reactant-predicting extension used as second baseline for comparison; expected
    average AUROC 0.53
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:3645849a2debd89e8da1f223dab3afcb360de23582506900c77f9071f577361f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# enzyme-specificity-model-evaluation

## Summary

Evaluates in silico reactant-prediction models for the nine major human cytochrome P450 enzymes (CYP1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) by per-enzyme cross-validation AUROC on a 1632-molecule 'plausible reactant' dataset, benchmarked against ADMET Predictor and SMARTCyp.

## When to use

Use when classifying whether query molecules react with specific human CYP450 isoforms and you need per-enzyme AUROC evaluation, cross-validation, or benchmarking against ADMET Predictor/SMARTCyp baselines.

## When NOT to use

- The goal is metabolite structure or exact site-of-metabolism (regioselectivity) prediction rather than the first-task reactant/non-reactant classification that CypReact performs
- The query molecules are not plausible CYP450 reactants (e.g. outside drug-like chemical space covered by the 1632-molecule training/test dataset), where the learned models' applicability is unverified
- You only have prediction scores with no ground-truth reactant labels, making AUROC computation and cross-validation impossible

## Inputs

- Query molecules represented as structures compatible with CypReact input (plausible reactant molecules, e.g. SMILES/SDF as consumed by the tool)
- CypReact per-enzyme learned models and the 1632-molecule reactant dataset from the Bitbucket repository
- Optional: predictions from ADMET Predictor and SMARTCyp on the same molecule set for baseline comparison

## Outputs

- Per-enzyme binary reactant/non-reactant predictions for each of the nine CYP450 isoforms
- Per-enzyme cross-validation AUROC values and their average (expected 0.83–0.92)
- Benchmark table comparing CypReact against ADMET Predictor (avg AUROC 0.75) and SMARTCyp (avg AUROC 0.53)

## How to apply

Obtain CypReact from its Bitbucket repository (https://bitbucket.org/Leon_Ti/cypreact), which ships the learned per-enzyme models and the 1632-molecule reactant dataset. Run CypReact in cross-validation mode for each of the nine target enzymes (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) and compute the area under the ROC curve from the cross-validated reactant/non-reactant predictions per enzyme. Aggregate the nine per-enzyme AUROCs into an average and check that per-enzyme and average values fall in the reported 0.83–0.92 range. For benchmarking, run ADMET Predictor (expected average AUROC 0.75) and the reactant-predicting extension of SMARTCyp (expected average AUROC 0.53) on the same molecules and confirm CypReact outperforms both. For a stricter estimate, apply the learned models to a previously unseen molecule set, where the paper reports CypReact performs even better than in cross-validation while still surpassing both baselines.

## Related tools

- **CypReact** (Primary reactant-prediction tool whose per-enzyme learned models are run in cross-validation mode to produce AUROC-scored reactant predictions for the nine CYP450 enzymes) — https://bitbucket.org/Leon_Ti/cypreact
- **ADMET Predictor** (Baseline reactant-prediction tool used for comparison; expected average AUROC 0.75 versus CypReact's 0.83-0.92)
- **SMARTCyp** (Reactant-predicting extension used as second baseline for comparison; expected average AUROC 0.53)

## Evaluation signals

- Per-enzyme cross-validation AUROC for each of the nine CYP450 isoforms falls within the reported 0.83-0.92 range, and the computed average AUROC does as well
- CypReact's average AUROC exceeds ADMET Predictor's reported average of 0.75 and SMARTCyp's reported average of 0.53 on the same molecule set
- Results table contains one AUROC entry per enzyme for all nine isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) computed over the full 1632-molecule reactant dataset
- Holdout evaluation on a previously unseen molecule set yields AUROCs at least as high as the cross-validation values, mirroring the paper's finding that CypReact 'did even better' on unseen molecules

## Limitations

- Reported AUROC range (0.83-0.92) is from cross-validation on 1632 molecules all deemed 'plausible' reactants; performance on molecules outside this reactant space is not established
- Baseline comparisons depend on ADMET Predictor and SMARTCyp being run on the same data; differing preprocessing could change the reported gaps (0.75 and 0.53 average AUROCs)
- CypReact addresses only the first task of reactant prediction per CYP450 isoform; downstream metabolite identity prediction is out of scope

## Evidence
