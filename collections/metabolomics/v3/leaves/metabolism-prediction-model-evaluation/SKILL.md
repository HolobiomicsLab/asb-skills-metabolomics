---
name: metabolism-prediction-model-evaluation
description: Use when evaluating or comparing reactant-prediction performance for
  CYP450 enzymes (CYP1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) on molecule-enzyme
  pairs with binary reactant labels, using AUROC as the metric.
when_to_use_negative:
- The task is site-of-metabolism (SOM) prediction or metabolite structure generation
  rather than binary reactant/non-reactant classification per molecule-enzyme pair
- ADMET Predictor must be freshly executed rather than compared via its reported average
  AUROC of 0.75, since it is a commercial tool that cannot be freely run
- The input dataset lacks binary reactant labels for the nine CYP450 enzymes, so AUROC
  cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2815
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_3324
tools:
- name: CypReact
  role: Tool under evaluation; provides cross-validation AUROC reference range of
    0.83-0.92 on the 1632-molecule dataset and is runnable from its Bitbucket repository
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: SMARTCyp
  role: Baseline comparator; its reactant-predicting extension is executed over molecule-enzyme
    pairs to compute the average AUROC under evaluation
- name: ADMET Predictor
  role: Commercial baseline comparator; its reported average AUROC of 0.75 is recorded
    for comparison since it cannot be freely executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:1c7827f693b56dcf4f1258fc51308f9adde5dec2f7b735add2a743363f0645fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2815
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_3324
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolism-prediction-model-evaluation

## Summary

Benchmarks in silico CYP450 reactant-prediction tools (CypReact, ADMET Predictor, SMARTCyp) by computing per-enzyme and average AUROC over a 1632-molecule benchmark spanning nine human cytochrome P450 enzymes, to establish whether a new tool surpasses published baselines.

## When to use

Use when evaluating or comparing reactant-prediction performance for CYP450 enzymes (CYP1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4) on molecule-enzyme pairs with binary reactant labels, using AUROC as the metric.

## When NOT to use

- The task is site-of-metabolism (SOM) prediction or metabolite structure generation rather than binary reactant/non-reactant classification per molecule-enzyme pair
- ADMET Predictor must be freshly executed rather than compared via its reported average AUROC of 0.75, since it is a commercial tool that cannot be freely run
- The input dataset lacks binary reactant labels for the nine CYP450 enzymes, so AUROC cannot be computed

## Inputs

- 1632-molecule reactant benchmark (SMILES strings or SDF records) with binary reactant labels for each of the nine CYP450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4)
- Per molecule-enzyme pair reactant scores from the tool under evaluation (e.g., SMARTCyp reactant scores)
- Optionally, a previously unseen set of molecules for out-of-sample testing

## Outputs

- Per-enzyme AUROC values for the nine CYP450 enzymes
- Average AUROC across the nine enzymes for the evaluated tool
- Comparison table recording evaluated and reported baselines (e.g., smartcyp_auroc_results.csv)

## How to apply

Load the 1632-molecule reactant dataset with binary reactant labels for each of the nine CYP450 enzymes, then run the candidate predictor (e.g., the reactant-predicting extension of SMARTCyp or CypReact from https://bitbucket.org/Leon_Ti/cypreact) over every molecule-enzyme pair to obtain a per-pair reactant score, mirroring the cross-validation protocol used for CypReact. Compute the AUROC separately for each of the nine enzymes from the scores and labels, then average the nine AUROC values. Judge the result against the published reference points: CypReact's cross-validation AUROC of 0.83-0.92, ADMET Predictor's average AUROC of 0.75, and SMARTCyp's average AUROC of 0.53. ADMET Predictor is commercial and cannot be freely executed, so record its reported value for expert-review comparison rather than recomputing it. Optionally apply the learned CypReact models to a previously unseen set of molecules to confirm that performance holds and still surpasses SMARTCyp and ADMET Predictor.

## Related tools

- **CypReact** (Tool under evaluation; provides cross-validation AUROC reference range of 0.83-0.92 on the 1632-molecule dataset and is runnable from its Bitbucket repository) — https://bitbucket.org/Leon_Ti/cypreact
- **SMARTCyp** (Baseline comparator; its reactant-predicting extension is executed over molecule-enzyme pairs to compute the average AUROC under evaluation)
- **ADMET Predictor** (Commercial baseline comparator; its reported average AUROC of 0.75 is recorded for comparison since it cannot be freely executed)

## Examples

```
python run_reactant_benchmark.py --dataset reactants_1632.sdf --tool smartcyp --enzymes CYP1A2,CYP2A6,CYP2B6,CYP2C8,CYP2C9,CYP2C19,CYP2D6,CYP2E1,CYP3A4 --out smartcyp_auroc_results.csv
```

## Evaluation signals

- The computed average AUROC for the reactant-predicting extension of SMARTCyp should match the reported value of 0.53
- The evaluated tool's average AUROC should fall below CypReact's reported cross-validation AUROC range of 0.83-0.92 to confirm CypReact's superiority
- Nine separate AUROC values (one per enzyme: CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) should be produced and their mean taken, rather than pooling molecule-enzyme pairs
- ADMET Predictor's value in the comparison table should equal the reported average of 0.75 (recorded, not recomputed)

## Limitations

- ADMET Predictor is commercial and cannot be freely executed, so its performance can only be compared via the reported average AUROC of 0.75
- The benchmark covers 1632 molecules each considered a 'plausible' reactant; results are specific to this dataset and the cross-validation protocol used for CypReact
- On a previously unseen set of molecules performance may differ, although the paper reports CypReact did even better out-of-sample and still significantly surpassed SMARTCyp and ADMET Predictor

## Evidence
