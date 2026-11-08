---
name: gaussian-process-meta-learning-prior-optimization
description: Use when you must project DNN-predicted retention times to a specific
  chromatographic method using only ~10 identified molecules with predicted-experimental
  RT pairs, while also getting credible intervals.
when_to_use_negative:
- The target chromatographic method has no identified molecules at all — the target-task
  update needs experimental-predicted RT pairs from ~10 known molecules
- You need a point RT prediction from chemical structure alone (that is the DNN regression
  task on SMRT, not the projection between chromatographic methods)
- Molecules in the dataset are non-retained (RT < 5 min) and were not excluded — including
  them distorts the learned prior and scaling
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: cmmrt
  role: Python package implementing the GP meta-training for projections between chromatographic
    methods; installed via 'make install' and driven with 'make train_projections'
    / 'make test_projections'
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Runtime environment for the cmmrt package scripts that meta-train the GP and
    compute posterior predictive projections
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:227d0fe60c7fee079f5044f6359896845d35a79e7329d6fabd1f4891c521bcc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - cmmrt
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gaussian-process-meta-learning-prior-optimization

## Summary

Meta-learns a GP prior over RT projection functions by minimizing negative Leave-One-Out log predictive probability across PredRet meta-tasks, then adapts it to a target chromatographic method from ~10 identified molecules by estimating only the residual variance.

## When to use

Use when you must project DNN-predicted retention times to a specific chromatographic method using only ~10 identified molecules with predicted-experimental RT pairs, while also getting credible intervals.

## When NOT to use

- The target chromatographic method has no identified molecules at all — the target-task update needs experimental-predicted RT pairs from ~10 known molecules
- You need a point RT prediction from chemical structure alone (that is the DNN regression task on SMRT, not the projection between chromatographic methods)
- Molecules in the dataset are non-retained (RT < 5 min) and were not excluded — including them distorts the learned prior and scaling

## Inputs

- PredRet meta-task datasets of predicted-vs-experimental RT pairs per chromatographic method (FEM long: 342, FEM orbitrap plasma: 133, LIFE old: 148, RIKEN: 271, remainder: 2418 molecules), with non-retained molecules excluded
- Target-task training set of ~10 stratified-sampled identified molecules with experimental RTs paired to DNN-predicted RTs
- DNN-predicted RTs for the candidate molecules to be projected

## Outputs

- Meta-learned GP model over projection functions saved to the saved_models folder
- Projected RTs for the target chromatographic method (posterior predictive mean)
- 95% credible intervals for each projected retention time
- Delta-L_avg metric comparing meta-learned GP against a non-meta-learned reference GP

## How to apply

Load PredRet meta-task datasets of predicted-vs-experimental RT pairs per chromatographic method (FEM long: 342 molecules, FEM orbitrap plasma: 133, LIFE old: 148, RIKEN: 271, remainder: 2418), excluding non-retained molecules (RT < 5 min). Transform all RTs to log space via log(1+x) and robustly scale inputs to [0,1] and outcomes to [-1,1] using the median and 0.741*IQR (chosen so sigma ≈ 0.741*IQR under normality), learning both scalings from the predicted RTs. Meta-learn the GP prior over projection functions with a constant mean function and a polynomial kernel of degree 4 — selected for the lowest variability in Delta-L_avg — by minimizing the negative Leave-One-Out log predictive probability across the meta-tasks. For the target method, update the prior with ~10 stratified-sampled identified molecules, estimating only the residual variance via type-II maximum likelihood for 250 epochs with Adam at learning rate 0.01. Compute the GP posterior predictive mean and 95% credible intervals to project predicted RTs to the target method, and save the resulting GP model to the saved_models folder.

## Related tools

- **cmmrt** (Python package implementing the GP meta-training for projections between chromatographic methods; installed via 'make install' and driven with 'make train_projections' / 'make test_projections') — https://github.com/constantino-garcia/cmmrt
- **Python** (Runtime environment for the cmmrt package scripts that meta-train the GP and compute posterior predictive projections) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make train_projections  # then remove --smoke_test from the Makefile rule for full meta-training; results saved to saved_models
```

## Evaluation signals

- The meta-learned GP must achieve Delta-L_avg > 0, i.e. higher predictive marginal log-likelihood than a non-meta-learned reference GP with constant mean and squared exponential kernel trained directly on the target task
- Projection MAE against the four reference CMs from the PredRet database (FEM long, LIFE old, FEM orbitrap plasma, RIKEN) is competitive with previous approaches despite using as few as 10 molecules
- make test_projections produces CSVs and figures summarizing the projections under results/projection with finite posterior means and valid 95% credible intervals
- Robust scaling invariants: inputs lie in [0,1] and outcomes in [-1,1] after scaling with the median and 0.741*IQR learned from the predicted RTs
- Only the residual variance is re-estimated on the target task (250 Adam epochs, lr 0.01); kernel hyperparameters come from meta-training, so a diff of the GP should show unchanged mean function (constant) and polynomial degree-4 kernel

## Limitations

- Works from as few as 10 identified molecules, but performance degrades gracefully rather than matching methods trained on full datasets; competitive error is the goal, not superiority at large sample sizes
- The polynomial kernel of degree 4 with constant mean was selected by lowest variability in Delta-L_avg on PredRet meta-tasks; transfer to very different chromatographic-space geometries is not validated
- Robust scaling constants (median, 0.741*IQR) and the log(1+x) transform are learned from the predicted RTs, so changes to the upstream DNN predictor require relearning the scalings
- Repository notes that for reproducing paper results the 'paper' branch must be used, since code continues to be developed for CEU Mass Mediator integration

## Evidence

- [methods] Meta-learning of GP prior over projection functions: "meta-tasks are used to learn a useful prior distribution, which is then used as starting point to solve the target-task."
- [abstract] Few-shot projection capability: [evidence span withheld — anchor status: not_found]
- [intro] Credible intervals advantage: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT) projections between CMs, and not only a point"
- [readme] Validation protocol: "Four CMs of reference are used for the test: FEM long, LIFE old, FEM orbitrap plasma and RIKEN. For each system, the GP is meta-trained using all other CMs from the PredRet database before testing"
- [readme] Repository rule for meta-training: "make train_projections: Meta-trains a Gaussian Process (GP) for computing projections between CMs using the PredRet database for a few epochs. The resulting GP is stored in the saved_models folder."
