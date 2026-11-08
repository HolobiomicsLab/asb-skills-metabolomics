---
name: retention-time-projection-meta-learning
description: Use when you have a small set (10–50 molecules) of paired predicted/experimental
  RTs for a target chromatographic method and need to map DNN-predicted RTs onto that
  method's scale with 95% prediction intervals.
when_to_use_negative:
- The task is training the RT predictor itself on the SMRT dataset from fingerprints/descriptors
  — use the predictor-training workflow (make train_predictor), not projection.
- No predicted RTs from the SMRT-trained DNN exist for the compounds and no experimental
  RTs for a few known molecules are available — the projection needs paired predicted/experimental
  values.
- Fragmentation spectra are available for candidate metabolites — standard annotation
  with spectral matching is preferable; this skill targets the case where annotation
  must rely on just m/z ratios and RTs.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: cmmrt
  role: Python package implementing GP meta-training and RT projection between chromatographic
    methods; exposes make targets train_projections/test_projections and the projections_to_different_cm.ipynb
    notebook
  repo: https://github.com/constantino-garcia/cmmrt
- name: PredRet
  role: Database of experimental RTs across chromatographic methods used as the meta-training
    tasks and as the leave-one-CM-out validation targets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:bce124f84fccc480f613363a8c947bff761615b21d92b195876534ff753fa023
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - PredRet
  - Python
  - cmmrt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# retention-time-projection-meta-learning

## Summary

A Bayesian meta-learning approach that projects DNN-predicted retention times (from a fingerprint model trained on the 80,038-experiment SMRT dataset) onto a specific chromatographic method's experimental RT scale using as few as 10 identified molecules, with calibrated confidence intervals.

## When to use

Use when you have a small set (10–50 molecules) of paired predicted/experimental RTs for a target chromatographic method and need to map DNN-predicted RTs onto that method's scale with 95% prediction intervals.

## When NOT to use

- The task is training the RT predictor itself on the SMRT dataset from fingerprints/descriptors — use the predictor-training workflow (make train_predictor), not projection.
- No predicted RTs from the SMRT-trained DNN exist for the compounds and no experimental RTs for a few known molecules are available — the projection needs paired predicted/experimental values.
- Fragmentation spectra are available for candidate metabolites — standard annotation with spectral matching is preferable; this skill targets the case where annotation must rely on just m/z ratios and RTs.

## Inputs

- Pairs of predicted RTs (from the fingerprint-trained DNN, projected to log space) and experimental RTs of 10–50 identified molecules in the target chromatographic method
- PredRet database chromatographic methods used as meta-training tasks
- Predicted RT database built with the SMRT-trained DNN (alvaDesc ECFP, MACCSFP and PFP fingerprints)
- CSV of compounds (e.g. pubchem id with SMILES/InChI) if regenerating fingerprints upstream

## Outputs

- projection_results.csv with MAE, MEDAE, median relative error and scaled interval score per CM and training-point count
- Projected RTs with 95% prediction intervals for held-out target test molecules
- Meta-trained GP prior stored in the saved_models folder
- Figures and CSVs summarizing projections under results/projection

## How to apply

Meta-train a GP prior over projection functions on the meta-task chromatographic methods from the PredRet database by minimizing the negative Leave-One-Out log predictive probability, using a GP with a constant mean and a polynomial kernel of degree 4 (chosen for its lowest variability of ΔL_avg). Preprocess paired predicted/experimental RTs with a log(1+x) transform followed by robust median/IQR scaling (0.741 IQR factor) learned from predicted RTs. For the target method, fit the GP posterior by maximizing type II likelihood (Adam, learning rate 0.01) on the 10–50 known-molecule pairs, then compute projected RTs with 95% prediction intervals for held-out test molecules. Validate with a leave-one-chromatographic-method-out protocol using FEM long (342 molecules), FEM orbitrap plasma (133), LIFE old (148) and RIKEN (271) as target CMs — excluding non-retained molecules (RT < 5 min) — building target training sets by stratified subsampling of 10, 20, 25, 30, 40 and 50 known molecules, repeated 10 times.

## Related tools

- **cmmrt** (Python package implementing GP meta-training and RT projection between chromatographic methods; exposes make targets train_projections/test_projections and the projections_to_different_cm.ipynb notebook) — https://github.com/constantino-garcia/cmmrt
- **PredRet** (Database of experimental RTs across chromatographic methods used as the meta-training tasks and as the leave-one-CM-out validation targets)

## Examples

```
make install && make test_projections  # reproduces projections to FEM long, LIFE old, FEM orbitrap plasma and RIKEN; results in results/projection
```

## Evaluation signals

- projection_results.csv reports MAE, MEDAE, median relative error and scaled interval score for each target CM and each training-point count (10–50 molecules), with 10 subsample repeats.
- With only ~10 known molecules the projections still achieve competitive error rates versus previous approaches, as claimed for the Bayesian meta-learning approach.
- Meta-training with the constant-mean, degree-4 polynomial kernel yields the lowest variability of ΔL_avg across meta-tasks.
- The 95% prediction intervals are reported alongside point projections (scaled interval score is finite and shrinks as training points increase from 10 to 50).
- Running make test_projections reproduces CSVs and figures under results/projection for the four reference CMs (FEM long, LIFE old, FEM orbitrap plasma, RIKEN).

## Limitations

- The repository continues to be developed for CEU Mass Mediator integration, so the branch 'paper' must be used to reproduce the published results; Makefile targets run smoke tests (few epochs, subsampled data) unless the --smoke_test flag is removed.
- Non-retained molecules (RT < 5 min) are excluded from the target CM training sets, so projections target retained compounds only.
- Meta-training depends on the PredRet database of chromatographic methods; methods outside that space (meta-tasks) may not be well covered by the learned GP prior.
- Generating fingerprints for new compounds requires the licensed alvaDesc software (the RDKit notebook exists but the paper's DNN used alvaDesc fingerprints).

## Evidence

- [methods] Meta-learning of GP prior over projection functions: "meta-tasks are used to learn a useful prior distribution, which is then used as starting point to solve the target-task."
- [intro] Bayesian projections provide intervals from few molecules: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT) projections between CMs"
- [abstract] As few as 10 identified molecules suffice: "it is enough that just as few as 10 molecules of a given experiment have been identified"
- [readme] cmmrt make target meta-trains GP on PredRet: "Meta-trains a Gaussian Process (GP) for computing projections between CMs using the PredRet database"
- [readme] Leave-one-CM-out validation targets in cmmrt: "Four CMs of reference are used for the test: FEM long, LIFE old, FEM orbitrap plasma and RIKEN."
