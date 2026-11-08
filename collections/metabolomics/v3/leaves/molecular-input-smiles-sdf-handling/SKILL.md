---
name: molecular-input-smiles-sdf-handling
description: Use when the input is a molecular dataset (SMILES strings or SDF records)
  that must be loaded with per-enzyme binary reactant labels and scored by CypReact
  for in silico CYP450 reactant prediction.
when_to_use_negative:
- The goal is site-of-metabolism prediction (atom-level which-site output) rather
  than molecule-level reactant classification — SMARTCyp addresses the site question,
  CypReact addresses the reactant question
- The input is not molecular structural data (SMILES/SDF) — e.g. omics or spectral
  data unrelated to reactant prediction
- No binary reactant labels and no cross-validation protocol are available, so AUROC
  cannot be computed to judge correct application
tools:
- name: CypReact
  role: 'Primary predictor: loads the SMILES/SDF molecules and scores reactant probability
    per CYP450 enzyme using its learned per-enzyme models; run in cross-validation
    mode for evaluation'
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: SMARTCyp
  role: 'Baseline tool: a reactant-predicting extension scored over the same molecule-enzyme
    pairs to obtain a reference average AUROC of 0.53 for comparison against CypReact'
- name: ADMET Predictor
  role: 'Commercial baseline: its reported average AUROC of 0.75 is recorded in the
    results table for expert-review comparison'
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:1374443e0ea2763cf1ce3acc7af67c440a5eb9cff6162be94dc1ec3b22777d1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADMET Predictor
  - CypReact
  - SMARTCyp
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-input-smiles-sdf-handling

## Summary

Loads and standardizes molecular inputs for CYP450 reactant prediction: molecules supplied as SMILES strings or SDF records are paired with binary reactant labels for the nine major human CYP450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) so they can be scored by CypReact and evaluated by AUROC.

## When to use

Use when the input is a molecular dataset (SMILES strings or SDF records) that must be loaded with per-enzyme binary reactant labels and scored by CypReact for in silico CYP450 reactant prediction.

## When NOT to use

- The goal is site-of-metabolism prediction (atom-level which-site output) rather than molecule-level reactant classification — SMARTCyp addresses the site question, CypReact addresses the reactant question
- The input is not molecular structural data (SMILES/SDF) — e.g. omics or spectral data unrelated to reactant prediction
- No binary reactant labels and no cross-validation protocol are available, so AUROC cannot be computed to judge correct application

## Inputs

- SMILES strings for 1632 'plausible' reactant molecules
- SDF records for the same 1632-molecule reactant dataset
- Binary reactant labels (reactant / non-reactant) per molecule for each of the nine CYP450 enzymes
- CypReact package with learned per-enzyme models from https://bitbucket.org/Leon_Ti/cypreact

## Outputs

- Per-molecule reactant scores for each of the nine CYP450 enzymes
- Per-enzyme AUROC values plus a nine-enzyme average AUROC results table
- Results file (e.g. smartcyp_auroc_results.csv) recording per-enzyme AUROCs and baseline comparisons

## How to apply

Obtain the CypReact package from its public Bitbucket repository, which ships the learned per-enzyme models and the 1632-molecule 'plausible reactant' dataset. Load the molecules as SMILES strings or SDF records and attach binary reactant labels for each of the nine CYP450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4). Run CypReact in cross-validation mode per enzyme (or score a previously unseen molecule set with the pre-trained models) to obtain a reactant score for every molecule-enzyme pair. Compute the AUROC per enzyme from the scores and labels, then average the nine per-enzyme AUROC values into a results table. Validate that the per-enzyme and average AUROC values fall within the reported cross-validation range of 0.83-0.92.

## Related tools

- **CypReact** (Primary predictor: loads the SMILES/SDF molecules and scores reactant probability per CYP450 enzyme using its learned per-enzyme models; run in cross-validation mode for evaluation) — https://bitbucket.org/Leon_Ti/cypreact
- **SMARTCyp** (Baseline tool: a reactant-predicting extension scored over the same molecule-enzyme pairs to obtain a reference average AUROC of 0.53 for comparison against CypReact)
- **ADMET Predictor** (Commercial baseline: its reported average AUROC of 0.75 is recorded in the results table for expert-review comparison)

## Examples

```
git clone https://bitbucket.org/Leon_Ti/cypreact  # then run CypReact in cross-validation mode per enzyme (CYP1A2...CYP3A4) on the 1632-molecule SMILES/SDF dataset
```

## Evaluation signals

- Per-enzyme cross-validation AUROC values across all nine CYP450s (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) fall within the reported 0.83-0.92 range
- The nine-enzyme average AUROC falls within the reported 0.83-0.92 range
- The dataset contains exactly 1632 molecules, each considered a 'plausible' reactant
- SMARTCyp baseline average AUROC matches the reported value of 0.53 and ADMET Predictor's reported average is 0.75, both below CypReact's range
- The results table includes one AUROC entry per enzyme with no missing molecule-enzyme pairs

## Limitations

- The benchmark dataset is limited to 1632 molecules each deemed a 'plausible' reactant, so performance on implausible or out-of-domain structures is not established by this protocol
- ADMET Predictor is commercial and cannot be freely executed, so its 0.75 average AUROC can only be recorded as a reported baseline rather than reproduced
- Cross-validation AUROC estimates performance on data seen during model development; performance on a previously unseen set must be evaluated separately (where the article reports CypReact did even better)

## Evidence

- [results] CypReact achieves cross-validation AUROC values of 0.83-0.92 across the nine CYP450 enzymes: [evidence span withheld — anchor status: not_found]
- [methods] Load the 1632-molecule reactant dataset: [evidence span withheld — anchor status: not_found]
