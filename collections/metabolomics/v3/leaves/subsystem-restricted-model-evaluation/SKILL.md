---
name: subsystem-restricted-model-evaluation
description: Use when you need to test whether a constraint-based MPI predictor (SARTRE)
  generalizes to metabolite-protein pairs from metabolic subsystems that were excluded
  from training, or to assess subsystem-level accuracy drops.
when_to_use_negative:
- You want the standard full-network evaluation; use evaluate.py in each gold standard
  directory instead
- Your goal is cross-organism evaluation excluding shared metabolite-protein pairs;
  use shared.py rather than subsys.py
- Features other than shadow prices (e.g. 128-bit RDKit molecular fingerprints) are
  the target feature set for this evaluation
tools:
- name: SARTRE
  role: Framework whose random forest classifier is retrained under the subsystem-exclusion
    protocol
  repo: https://github.com/fayazsoleymani/SARTRE
- name: scikit-learn
  role: Provides the random forest classifier and evaluation metrics used by subsys.py
- name: GUROBI
  role: Optimizer that produced the shadow-price features via optimization.py
- name: COBRAToolbox
  role: Used in MATLAB to curate iJO1366 into ModelIrrevOpt90.mat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:990f691bc66249b5d6a18ada738e492605213a6c91b87bef617466ccc2ffc59d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fayazsoleymani/SARTRE
  tools:
  - COBRAToolbox
  - GUROBI
  - Python
  - SARTRE
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Subsystem-restricted model evaluation

## Summary

Evaluates SARTRE metabolite-protein interaction (MPI) classifiers when trained on all iJO1366 pairs except those from a specific metabolic subsystem (Alternate Carbon Metabolism or Cofactor and Prosthetic Group Biosynthesis), quantifying how much shadow-price coverage of a subsystem drives prediction accuracy.

## When to use

Use when you need to test whether a constraint-based MPI predictor (SARTRE) generalizes to metabolite-protein pairs from metabolic subsystems that were excluded from training, or to assess subsystem-level accuracy drops.

## When NOT to use

- You want the standard full-network evaluation; use evaluate.py in each gold standard directory instead
- Your goal is cross-organism evaluation excluding shared metabolite-protein pairs; use shared.py rather than subsys.py
- Features other than shadow prices (e.g. 128-bit RDKit molecular fingerprints) are the target feature set for this evaluation

## Inputs

- Curated irreversible iJO1366 model file (ModelIrrevOpt90.mat)
- Pre-computed shadow prices for the stitch_ecoli gold standard at confidence score 400
- Subsystem annotation of iJO1366 metabolite-protein pairs (acm, cpgb)

## Outputs

- Accuracy of a random forest model on held-out Alternate Carbon Metabolism pairs (~0.615)
- Accuracy of a random forest model on held-out Cofactor and Prosthetic Group Biosynthesis pairs (~0.815)

## How to apply

Load the curated irreversible iJO1366 model (ModelIrrevOpt90.mat, created by modelPreparing_iJO1366.m with COBRAToolbox in MATLAB) and the pre-computed shadow prices from the stitch_ecoli gold standard at the medium confidence score of 400, optimized with GUROBI and preprocessed with cleaning.py. Exclude all metabolite-protein pairs belonging to the target subsystem — 'Alternate Carbon Metabolism' (acm) or 'Cofactor and Prosthetic Group Biosynthesis' (cpgb) — and train a scikit-learn random forest classifier on the remaining pairs using shadow prices as metabolite features. Evaluate each trained model on the held-out STITCH-E.coli (400) test set restricted to the excluded subsystem and record accuracy. Rationale: shadow prices covering a metabolic system carry the discriminative signal, so excluding pairs from a subsystem inflates false positives and depresses accuracy for that subsystem.

## Related tools

- **SARTRE** (Framework whose random forest classifier is retrained under the subsystem-exclusion protocol) — https://github.com/fayazsoleymani/SARTRE
- **scikit-learn** (Provides the random forest classifier and evaluation metrics used by subsys.py)
- **GUROBI** (Optimizer that produced the shadow-price features via optimization.py)
- **COBRAToolbox** (Used in MATLAB to curate iJO1366 into ModelIrrevOpt90.mat)

## Examples

```
cd subsys_shared; python3 subsys.py acm; python3 subsys.py cpgb
```

## Evaluation signals

- The acm-excluded model achieves accuracy of approximately 0.615 on the STITCH-E.coli (400) test set
- The cpgb-excluded model achieves accuracy of approximately 0.815 on the STITCH-E.coli (400) test set
- Accuracy for the excluded subsystem should be lower than the full-network baseline because the drop is attributed to increased false positives
- Training must use shadow-price features at the medium STITCH confidence score of 400, not low (150), high (700), or highest (900)

## Limitations

- Findings indicate shadow prices covering different metabolic systems increase prediction accuracy; restricted accuracy drops (e.g. 0.615 for Alternate Carbon Metabolism) reflect larger numbers of false positives, so subsystem-level predictions should be interpreted cautiously
- Analysis was only demonstrated for two E. coli subsystems (acm and cpgb) with the stitch_ecoli gold standard at confidence 400
- GUROBI requires an active license before shadow prices can be computed

## Evidence

- [readme] Subsystem exclusion protocol: "Second, we excluded metabolite-protein pairs of two subsystems of *E. coli* separately from stitch_ecoli and trained two models on remaining pairs."
- [readme] Commands: "These subsystems are Alternate Carbon Metabolism and Cofactor and Prosthetic Group Biosynthesis. This process can be done by executing:"
- [methods] Evaluation design rationale: "evaluation has been done by excluding shared metabolite-protein pairs, which exist in two GEMs, from the datasets, and training two separate models in remaining pairs"
- [results] Reported accuracies: [evidence span withheld — anchor status: not_found]
