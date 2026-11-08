---
name: stratified-molecule-subampling-per-chromatographic-method
description: Use when validating few-shot RT projection to a chromatographic method
  with limited identified molecules — e.g. testing meta-learned GP projections at
  10, 20, 25, 30, 40, 50 training points.
when_to_use_negative:
- The target chromatographic method has few identified molecules overall or contains
  non-retained molecules (RT < 5 min) that must be excluded before subsampling
- You need full-dataset projection accuracy rather than few-shot (10–50 molecule)
  evaluation
- Predicted and experimental RTs cannot be paired per molecule (no common compound
  identifiers)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: cmmrt
  role: Python package implementing the leave-one-CM-out projection experiment including
    stratified subsampling of target training sets and results written to CSV
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Runtime for the cmmrt projection experiment scripts
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:5e7fd7e29a71586c0e6be70efbad1fd6680457307cd061a3af6ba5a8efcdbdef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - cmmrt
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# stratified-molecule-subampling-per-chromatographic-method

## Summary

Builds target-task training sets for RT projection evaluation by repeatedly subsampling a small, stratified set of known molecules (10–50) from a held-out chromatographic method in PredRet, so that few-shot projection performance can be averaged over subsample repeats.

## When to use

Use when validating few-shot RT projection to a chromatographic method with limited identified molecules — e.g. testing meta-learned GP projections at 10, 20, 25, 30, 40, 50 training points.

## When NOT to use

- The target chromatographic method has few identified molecules overall or contains non-retained molecules (RT < 5 min) that must be excluded before subsampling
- You need full-dataset projection accuracy rather than few-shot (10–50 molecule) evaluation
- Predicted and experimental RTs cannot be paired per molecule (no common compound identifiers)

## Inputs

- PredRet experimental RTs for each chromatographic method (e.g. FEM long, FEM orbitrap plasma, LIFE old, RIKEN)
- DNN-predicted RTs from the SMRT-trained fingerprint model, paired by molecule
- List of target training-set sizes (10, 20, 25, 30, 40, 50) and number of repeats (10)

## Outputs

- Repeated stratified subsamples of paired predicted/experimental RTs per chromatographic method
- Per-method, per-training-size error metrics (MAE, MEDAE, median relative error, scaled interval score) written to projection_results.csv

## How to apply

For each target chromatographic method (e.g. FEM long, FEM orbitrap plasma, LIFE old, RIKEN with 342, 133, 148 and 271 molecules), exclude non-retained molecules first, then form paired predicted/experimental RT training sets by stratified subsampling at sizes 10, 20, 25, 30, 40 and 50 known molecules. Repeat each subsample 10 times so that error estimates (MAE, MEDAE, median relative error, scaled interval score) are averaged over repeated draws rather than a single split. Preprocess each paired set with the log(1+x) transform followed by robust median/IQR scaling (0.741 factor) learned from the DNN's predicted RTs. The rationale is to emulate the realistic scenario where only a handful of metabolites have been identified in the target experiment, while stratification guards against unrepresentative molecule draws.

## Related tools

- **cmmrt** (Python package implementing the leave-one-CM-out projection experiment including stratified subsampling of target training sets and results written to CSV) — https://github.com/constantino-garcia/cmmrt
- **Python** (Runtime for the cmmrt projection experiment scripts) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make test_projections
```

## Evaluation signals

- Subsample sizes match exactly 10, 20, 25, 30, 40 and 50 known molecules per target CM, each repeated 10 times
- Non-retained molecules (RT < 5 min) are absent from every subsample
- Each subsample yields paired predicted/experimental RTs preprocessed with log(1+x) and robust median/IQR scaling (0.741 factor)
- Leave-one-CM-out protocol holds: GP is meta-trained only on the remaining PredRet CMs, never on the target CM's test molecules
- Aggregate errors (MAE, MEDAE, median relative error, scaled interval score) decrease as training points increase from 10 to 50 and are written to projection_results.csv

## Limitations

- Only four target CMs (FEM long, FEM orbitrap plasma, LIFE old and RIKEN) were used for leave-one-out validation, so generalization to other methods is untested
- With only 10 training molecules the GP posterior may be unstable; results are averaged over 10 subsample repeats to mitigate this
- The README smoke tests run meta-training for only a few epochs; full-fidelity reproduction requires removing the smoke-test flag

## Evidence

- [methods] apply a leave-one-chromatographic-method-out protocol with FEM long, FEM orbitrap plasma, LIFE old and RIKEN (342, 133, 148 and 271 molecules) as target CMs: [evidence span withheld — anchor status: not_found]
- [methods] build target training sets by stratified subsampling of 10, 20, 25, 30, 40 and 50 known molecules, repeating each subsample 10 times: [evidence span withheld — anchor status: not_found]
- [readme] make test_projections: Tests the performance of GP meta-training for computing projections between CMs. Four CMs of reference are used for the test: "make test_projections: Tests the performance of GP meta-training for computing projections between CMs. Four CMs of reference are used for the test: FEM long, LIFE old, FEM orbitrap plasma and RIKEN"
- [abstract] a novel Bayesian meta-learning approach that can learn from just a few molecules: "a novel Bayesian meta-learning approach that can learn from just a few molecules"
