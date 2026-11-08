---
name: predictive-interval-quantification-for-rt-projections
description: Use when predicted RTs from a DNN (e.g. trained on SMRT) must be projected
  onto a target chromatographic method with few (~10) identified molecules and probabilistic
  intervals are needed to score candidate metabolites.
when_to_use_negative:
- The target chromatographic method has abundant identified molecules — a directly
  trained projection (non-meta-learned GP) may suffice and the meta-learned prior
  is unnecessary.
- Molecules are non-retained (RT < 5 min) and were not excluded; they distort the
  log-transformed, robustly scaled training pairs.
- Only point RT estimates are needed with no uncertainty requirement and no candidate
  ranking by z-score is planned.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: cmmrt
  role: Python package implementing meta-training of the GP prior over projection
    functions and computing posterior predictive means and 95% credible intervals;
    `make train_projections` / `make test_projections` run the experiments
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Execution environment for the cmmrt package's GP meta-learning and projection
    code
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:b65b9e625e7ef7cbc6e3756140aa03d2ee2040e08227f50c381faaa9c242328b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - cmmrt
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# predictive-interval-quantification-for-rt-projections

## Summary

Quantify uncertainty in retention-time projections between chromatographic methods using a Bayesian meta-learned Gaussian Process that yields a posterior predictive mean with 95% credible intervals, not just point estimates, from as few as 10 identified molecules.

## When to use

Use when predicted RTs from a DNN (e.g. trained on SMRT) must be projected onto a target chromatographic method with few (~10) identified molecules and probabilistic intervals are needed to score candidate metabolites.

## When NOT to use

- The target chromatographic method has abundant identified molecules — a directly trained projection (non-meta-learned GP) may suffice and the meta-learned prior is unnecessary.
- Molecules are non-retained (RT < 5 min) and were not excluded; they distort the log-transformed, robustly scaled training pairs.
- Only point RT estimates are needed with no uncertainty requirement and no candidate ranking by z-score is planned.

## Inputs

- PredRet database predicted-vs-experimental RT pairs per chromatographic method (meta-tasks)
- ~10 stratified-sampled identified molecules with experimental RTs for the target chromatographic method (target training points)
- Predicted RTs (e.g. from the SMRT-trained DNN regressor) to be projected

## Outputs

- GP posterior predictive mean projecting predicted RTs to the target chromatographic method
- 95% credible intervals for each projected RT (z-scores usable for candidate ranking)
- Serialized GP model saved to the saved_models folder

## How to apply

Load PredRet meta-task datasets of predicted-vs-experimental RT pairs per chromatographic method (FEM long: 342, FEM orbitrap plasma: 133, LIFE old: 148, RIKEN: 271, remainder: 2418), excluding non-retained molecules. Transform all RTs to log space via log(1+x) and robustly scale inputs to [0,1] and outcomes to [-1,1] using the median and 0.741*IQR (so sigma ~ 0.741*IQR under normality), learning both scalings from the predicted RTs. Meta-learn a GP prior over projection functions — constant mean function plus a polynomial kernel of degree 4, chosen for lowest variability in Delta-L_avg — by minimizing the negative Leave-One-Out log predictive probability across meta-tasks. For the target method, update this prior with ~10 stratified-sampled identified molecules, estimating only the residual variance via type-II maximum likelihood for 250 epochs with Adam at learning rate 0.01. The GP posterior predictive mean and 95% credible intervals give the projected RTs with uncertainty, which become z-scores for ranking candidates; validate that the meta-learned GP achieves Delta-L_avg > 0 versus a non-meta-learned GP with constant mean and squared exponential kernel trained directly on the target task, then save the model to the saved_models folder.

## Related tools

- **cmmrt** (Python package implementing meta-training of the GP prior over projection functions and computing posterior predictive means and 95% credible intervals; `make train_projections` / `make test_projections` run the experiments) — https://github.com/constantino-garcia/cmmrt
- **Python** (Execution environment for the cmmrt package's GP meta-learning and projection code) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make train_projections
```

## Evaluation signals

- Meta-learned GP must achieve Delta-L_avg > 0, i.e. higher predictive marginal log-likelihood than a non-meta-learned reference GP (constant mean, squared exponential kernel) trained directly on the target task.
- Projection quality is competitive using only ~10 stratified-sampled target training points (the paper reports competitive error rates from as few as 10 molecules).
- Only the residual variance is estimated on the target task (type-II ML, 250 epochs, Adam lr=0.01); all other hyperparameters come from the meta-learned prior.
- In downstream annotation, correct molecule ranks among top three candidates filtered by mass and ranked by z-scores in 68% of cases.
- Resulting GP model file appears in the saved_models folder after running train_projections.

## Limitations

- Requires identified molecules in the target experiment to build experimental-predicted RT pairs; without any, the projection cannot be fitted.
- Kernel choice (polynomial degree 4) was fixed by lowest Delta-L_avg variability on PredRet; other chromatographic systems may need re-validation.
- Only the residual variance is learned on the target task, so the meta-learned prior shape constrains the projection and may not adapt fully to atypical chromatographic methods.
- Non-retained molecules must be excluded from meta-task and target data, limiting coverage to retained compounds.

## Evidence

- [methods] meta-tasks are used to learn a useful prior distribution, which is then used as starting point to solve the target-task: "meta-tasks are used to learn a useful prior distribution, which is then used as starting point to solve the target-task"
- [intro] projections provide confidence intervals: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT) projections between CMs, and not only a point"
- [abstract] 10 molecules suffice for the projection: [evidence span withheld — anchor status: not_found]
- [readme] Meta-train GP for projections and save model: "Meta-trains a Gaussian Process (GP) for computing projections between CMs using the PredRet database for a few epochs. The resulting GP is stored in the `saved_models` folder."
- [abstract] correct molecule among top three ranked by z-scores: "our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
