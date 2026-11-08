---
name: cyp450-reactant-prediction
description: Use when a query molecule must be classified as a plausible reactant
  for one or more of the nine major human CYP450 enzymes, and predictions need AUROC-based
  benchmarking against published baselines (0.83-0.92 CypReact, 0.75 ADMET Predictor,
  0.53 SMARTCyp).
when_to_use_negative:
- The task is metabolite site-of-metabolism (SoM) localization rather than binary
  reactant/substrate classification — CypReact answers 'does this molecule react with
  a given CYP', not 'where on the molecule'.
- The target enzyme is not among the nine major human CYP450 isoforms covered by the
  learned models (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1,
  CYP3A4).
- A free, executable re-run of the ADMET Predictor baseline is required — it is commercial
  and only its reported average AUROC of 0.75 can be recorded as a reference.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CypReact
  role: 'Primary predictor: provides the learned per-enzyme reactant-classification
    models for the nine CYP450 isoforms and the 1632-molecule dataset; run in cross-validation
    mode to produce AUROC-benchmarked predictions.'
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: SMARTCyp
  role: 'Baseline comparator: the reactant-predicting extension of SMARTCyp is executed
    over the same 1632-molecule molecule-enzyme pairs under the same cross-validation
    protocol to reproduce its reported average AUROC of 0.53.'
- name: ADMET Predictor
  role: 'Literature baseline only: its reported average AUROC of 0.75 is recorded
    in the comparison table; the tool is commercial and not executed within this workflow.'
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:fad5a7b0b42aefa9827822c8d1cc880b40d03f8cd2d5cae36ed624d71966c853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cyp450-reactant-prediction

## Summary

In silico reactant prediction for the nine major human Cytochrome P450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) using the CypReact tool, benchmarked by cross-validation AUROC against SMARTCyp and ADMET Predictor.

## When to use

Use when a query molecule must be classified as a plausible reactant for one or more of the nine major human CYP450 enzymes, and predictions need AUROC-based benchmarking against published baselines (0.83-0.92 CypReact, 0.75 ADMET Predictor, 0.53 SMARTCyp).

## When NOT to use

- The task is metabolite site-of-metabolism (SoM) localization rather than binary reactant/substrate classification — CypReact answers 'does this molecule react with a given CYP', not 'where on the molecule'.
- The target enzyme is not among the nine major human CYP450 isoforms covered by the learned models (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4).
- A free, executable re-run of the ADMET Predictor baseline is required — it is commercial and only its reported average AUROC of 0.75 can be recorded as a reference.

## Inputs

- Query molecule structures (SMILES strings or SDF records) to be scored for CYP450 reactivity
- 1632-molecule reactant dataset with binary reactant labels for each of the nine CYP450 enzymes (shipped with the CypReact Bitbucket repository)
- Enzyme target specification (one of CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4)

## Outputs

- Per-molecule, per-enzyme reactant predictions/scores for the nine CYP450 isoforms
- Per-enzyme AUROC values from cross-validated predictions, expected in the 0.83-0.92 range
- Average AUROC across the nine enzymes plus a results table with baseline values (SMARTCyp 0.53, ADMET Predictor 0.75), e.g. written to smartcyp_auroc_results.csv

## How to apply

Obtain CypReact and its learned per-enzyme models from the public Bitbucket repository (https://bitbucket.org/Leon_Ti/cypreact), which also ships the 1632-molecule dataset of 'plausible' reactants. For each of the nine target enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4), run CypReact in cross-validation mode over the molecule-enzyme pairs and compute the area under the ROC curve (AUROC) from the cross-validated reactant predictions. Aggregate the nine per-enzyme AUROC values into an average and assemble a per-enzyme plus average results table, confirming values fall in the reported 0.83-0.92 range. For baseline comparison, run the reactant-predicting extension of SMARTCyp over the same molecule-enzyme pairs with the same cross-validation protocol, compute per-enzyme and average AUROC (expected average ~0.53), and record ADMET Predictor's reported average of 0.75 as a literature reference since ADMET Predictor is commercial and cannot be freely executed. Optionally, apply the learned CypReact models to a previously unseen molecule set, where the article reports CypReact performs even better than in cross-validation.

## Related tools

- **CypReact** (Primary predictor: provides the learned per-enzyme reactant-classification models for the nine CYP450 isoforms and the 1632-molecule dataset; run in cross-validation mode to produce AUROC-benchmarked predictions.) — https://bitbucket.org/Leon_Ti/cypreact
- **SMARTCyp** (Baseline comparator: the reactant-predicting extension of SMARTCyp is executed over the same 1632-molecule molecule-enzyme pairs under the same cross-validation protocol to reproduce its reported average AUROC of 0.53.)
- **ADMET Predictor** (Literature baseline only: its reported average AUROC of 0.75 is recorded in the comparison table; the tool is commercial and not executed within this workflow.)

## Evaluation signals

- Per-enzyme cross-validation AUROC values for all nine CYP450 isoforms (CYP1A2 through CYP3A4) fall within the reported 0.83-0.92 range.
- The computed nine-enzyme average AUROC exceeds the SMARTCyp baseline of 0.53 and the ADMET Predictor baseline of 0.75.
- The SMARTCyp reactant-predicting extension, run under the same cross-validation protocol on the same 1632-molecule dataset, reproduces an average AUROC of approximately 0.53.
- The results table contains one AUROC entry per enzyme (nine entries) plus an average, computed from cross-validated reactant predictions rather than a single train/test split.
- Applying the models to a previously unseen molecule set should yield performance at least as good as the cross-validation AUROC range, per the reported finding.

## Limitations

- The reported performance bounds are specific to the 1632-molecule 'plausible reactant' dataset; the article does not support extrapolating the 0.83-0.92 range to arbitrary custom molecule sets.
- ADMET Predictor (average AUROC 0.75) is commercial and cannot be freely executed, so its baseline number can only be compared, not reproduced, within this workflow.
- CypReact performs binary reactant prediction per enzyme pair; it does not output site-of-metabolism positions or metabolite structures.

## Evidence

- [abstract] Cross-validation AUROC of 0.83-0.92 on 1632 molecules: "Tests of CypReact, conducted over a data set of 1632 molecules (each considered a "plausible" reactant) show that it is very effective, with a (cross-validation) AUROC"
- [abstract] Outperforms ADMET Predictor and SMARTCyp baselines: "CypReact performs significantly better than other reactant prediction tools such as ADMET Predictor and (a reactant-predicting extension of) SMARTCyp, whose average AUROCs are 0.75 and 0.53"
