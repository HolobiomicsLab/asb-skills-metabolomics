---
name: auroc-computation-for-binary-enzyme-classification
description: Use when a molecule dataset carries binary reactant labels per CYP450
  enzyme and per-enzyme or averaged AUROC is needed to benchmark a reactant-prediction
  tool like CypReact, SMARTCyp, or ADMET Predictor.
when_to_use_negative:
- The goal is site-of-metabolism prediction (SMARTCyp's native atom-ranking output)
  rather than binary reactant classification with AUROC scoring.
- The dataset lacks binary reactant ground-truth labels per enzyme, so no AUROC can
  be computed.
- The tool to benchmark is ADMET Predictor, which is commercial and cannot be freely
  executed — record its reported average AUROC of 0.75 for expert-review comparison
  instead.
tools:
- name: SMARTCyp
  role: Reactant-predicting extension is run over every molecule-enzyme pair to produce
    reactant scores that are scored by AUROC
- name: CypReact
  role: Reference tool whose reported cross-validation AUROC range (0.83-0.92) and
    released models define the performance target being reproduced
  repo: https://bitbucket.org/Leon_Ti/cypreact
- name: ADMET Predictor
  role: Commercial baseline; its reported average AUROC of 0.75 is recorded in the
    comparison table but cannot be freely executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.8b00035
    title: 'CypReact: A Software Tool for in Silico Reactant Prediction for Human
      Cytochrome P450 Enzymes'
  source_content_hash: sha256:98f0851c8a71e5b1c654bb4d5ed850bff506c428fee7afea8a417d23fb5078fb
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
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# auroc-computation-for-binary-enzyme-classification

## Summary

Scores binary reactant-vs-nonreactant classification for the nine human CYP450 enzymes (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4) by computing per-enzyme AUROC and averaging, as used to benchmark CypReact against ADMET Predictor (average AUROC 0.75) and a reactant-predicting extension of SMARTCyp (average AUROC 0.53) on a 1632-molecule dataset.

## When to use

Use when a molecule dataset carries binary reactant labels per CYP450 enzyme and per-enzyme or averaged AUROC is needed to benchmark a reactant-prediction tool like CypReact, SMARTCyp, or ADMET Predictor.

## When NOT to use

- The goal is site-of-metabolism prediction (SMARTCyp's native atom-ranking output) rather than binary reactant classification with AUROC scoring.
- The dataset lacks binary reactant ground-truth labels per enzyme, so no AUROC can be computed.
- The tool to benchmark is ADMET Predictor, which is commercial and cannot be freely executed — record its reported average AUROC of 0.75 for expert-review comparison instead.

## Inputs

- 1632-molecule reactant dataset as SMILES strings or SDF records with binary reactant labels per CYP450 enzyme (CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4)
- Per molecule-enzyme pair reactant scores from the tool under test (e.g. SMARTCyp reactant-predicting extension scores)
- Reference baseline values (CypReact 0.83-0.92 cross-validation AUROC; ADMET Predictor 0.75; SMARTCyp extension 0.53) recorded for comparison

## Outputs

- Per-enzyme AUROC values for each of the nine CYP450 enzymes
- Average AUROC across the nine enzymes for the scored tool
- smartcyp_auroc_results.csv containing per-enzyme AUROCs, the tool average, and reference baseline values for expert-review comparison

## How to apply

Load the 1632-molecule reactant dataset (SMILES strings or SDF records) with binary reactant labels for each of the nine CYP450 enzymes. Run the tool under test (e.g. the reactant-predicting extension of SMARTCyp) over every molecule-enzyme pair to obtain a reactant score per pair, mirroring the cross-validation evaluation protocol used for CypReact. Compute the AUROC (area under the receiver operating characteristic curve) separately for each of the nine enzymes from the scores and labels, then average the nine AUROC values. Compare the computed average against the reported baselines: SMARTCyp extension at 0.53, ADMET Predictor at 0.75 (commercial, recorded for expert review rather than executed), and CypReact's 0.83-0.92 cross-validation range. Write the per-enzyme AUROCs, the tool average, and the reference baselines to a results file such as smartcyp_auroc_results.csv.

## Related tools

- **SMARTCyp** (Reactant-predicting extension is run over every molecule-enzyme pair to produce reactant scores that are scored by AUROC)
- **CypReact** (Reference tool whose reported cross-validation AUROC range (0.83-0.92) and released models define the performance target being reproduced) — https://bitbucket.org/Leon_Ti/cypreact
- **ADMET Predictor** (Commercial baseline; its reported average AUROC of 0.75 is recorded in the comparison table but cannot be freely executed)

## Examples

```
from sklearn.metrics import roc_auc_score; aurocs=[roc_auc_score(y, s) for y, s in per_enzyme_scores]; avg=sum(aurocs)/9; print(aurocs, avg)
```

## Evaluation signals

- The computed average AUROC for the reactant-predicting SMARTCyp extension should match the reported value of 0.53.
- The computed average AUROC should fall below CypReact's reported 0.83-0.92 cross-validation AUROC range, confirming CypReact's superiority.
- Per-enzyme AUROCs must be produced for all nine CYP450 enzymes before averaging; the table must contain nine per-enzyme values plus one average.
- ADMET Predictor's reported average AUROC of 0.75 should appear in the results table as a recorded reference, not as an executed measurement.
- Output file smartcyp_auroc_results.csv should contain per-enzyme AUROCs, the SMARTCyp average AUROC, and the reference baseline values.

## Limitations

- ADMET Predictor is commercial and cannot be freely executed; its 0.75 average AUROC must be recorded from the literature rather than reproduced.
- CypReact's 0.83-0.92 AUROC is a cross-validation figure; direct comparison assumes the same 1632-molecule dataset and evaluation protocol.
- The previously unseen external test set showed even better CypReact performance, so baseline comparisons on the 1632-molecule set are conservative for CypReact.

## Evidence

- [abstract] Tests of CypReact, conducted over a data set of 1632 molecules (each considered a "plausible" reactant): "Tests of CypReact, conducted over a data set of 1632 molecules (each considered a "plausible" reactant)"
- [abstract] Baseline AUROCs of SMARTCyp and ADMET Predictor: "CypReact performs significantly better than other reactant prediction tools such as ADMET Predictor and (a reactant-predicting extension of) SMARTCyp, whose average AUROCs are 0.75 and 0.53, "
- [other] Per-enzyme AUROC computation workflow: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion for the reproduced baseline: [evidence span withheld — anchor status: not_found]
- [abstract] Unseen-set performance of CypReact: "We then applied the learned CypReact models to a previously unseen set of molecules and found that our CypReact did even better"
