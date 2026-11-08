---
name: retention-time-projection-between-chromatographic-methods
description: Use when predicted RTs from a machine learning model must be projected
  to a specific chromatographic method for which only ~10 experimentally identified
  molecules (predicted-vs-experimental RT pairs) are available, and confidence intervals
  on the projections are needed.
when_to_use_negative:
- The target chromatographic method has no identified molecules at all — the GP prior
  must be updated with target-task training points to fit the projection.
- The goal is RT prediction itself from molecular structure; use the SMRT-trained
  DNN predictor skill instead of the projection module.
- Non-retained molecules (RT < 5 minutes) are part of the target-task data and must
  be modeled — these are excluded from the projection meta-tasks.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: cmmrt
  role: Python package implementing the Bayesian meta-learning GP projection; provides
    train_projections (meta-train GP on PredRet) and test_projections (test on FEM
    long, LIFE old, FEM orbitrap plasma, RIKEN); use branch `paper` to reproduce paper
    results
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Implementation language for the cmmrt package and the meta-learned GP projection
    workflow
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:32218e67ea0a10c9c76bef8aeb30a347650e9602374a228280d02aef5619fafd
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
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# retention-time-projection-between-chromatographic-methods

## Summary

A Bayesian meta-learning approach that maps predicted retention times (from a DNN trained on the SMRT dataset) to a target chromatographic method's experimental RT scale using as few as 10 identified molecules, via a meta-learned GP prior over projection functions that also yields 95% credible intervals.

## When to use

Use when predicted RTs from a machine learning model must be projected to a specific chromatographic method for which only ~10 experimentally identified molecules (predicted-vs-experimental RT pairs) are available, and confidence intervals on the projections are needed.

## When NOT to use

- The target chromatographic method has no identified molecules at all — the GP prior must be updated with target-task training points to fit the projection.
- The goal is RT prediction itself from molecular structure; use the SMRT-trained DNN predictor skill instead of the projection module.
- Non-retained molecules (RT < 5 minutes) are part of the target-task data and must be modeled — these are excluded from the projection meta-tasks.

## Inputs

- PredRet database meta-tasks: predicted-vs-experimental RT pairs per chromatographic method (FEM long: 342, FEM orbitrap plasma: 133, LIFE old: 148, RIKEN: 271, remainder: 2418 molecules)
- Predicted RTs from a DNN RT predictor trained on the 80,038-experimental-RT SMRT dataset
- ~10 stratified-sampled identified molecules from the target chromatographic method (experimental RTs paired with predicted RTs)

## Outputs

- GP posterior predictive mean projecting predicted RTs to the target method's experimental RT scale
- 95% credible intervals for each projected RT (enabling z-score ranking of annotation candidates)
- Saved GP projection model in the saved_models folder

## How to apply

Load the PredRet meta-task datasets of predicted-vs-experimental RT pairs per chromatographic method (FEM long: 342 molecules, FEM orbitrap plasma: 133, LIFE old: 148, RIKEN: 271, remainder: 2418), excluding non-retained molecules. Transform all RTs to log space via log(1+x) and robustly scale inputs to [0,1] and outcomes to [-1,1] using the median and 0.741*IQR (chosen so sigma ~ 0.741*IQR under normality), learning both scalings from the predicted RTs. Meta-learn the GP prior over projection functions with a constant mean function and a polynomial kernel of degree 4 (selected for lowest variability in Delta-L_avg) by minimizing the negative Leave-One-Out log predictive probability across the meta-tasks. For the target method, update the prior with the ~10 stratified-sampled identified molecules, estimating only the residual variance via type-II maximum likelihood for 250 epochs with Adam at learning rate 0.01, then compute the GP posterior predictive mean and 95% credible intervals and save the GP model to the saved_models folder.

## Related tools

- **cmmrt** (Python package implementing the Bayesian meta-learning GP projection; provides train_projections (meta-train GP on PredRet) and test_projections (test on FEM long, LIFE old, FEM orbitrap plasma, RIKEN); use branch `paper` to reproduce paper results) — https://github.com/constantino-garcia/cmmrt
- **Python** (Implementation language for the cmmrt package and the meta-learned GP projection workflow) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make train_projections
```

## Evaluation signals

- The meta-learned GP must achieve Delta-L_avg > 0, i.e. higher predictive marginal log-likelihood than a non-meta-learned reference GP (constant mean, squared exponential kernel) trained directly on the target task.
- Projection accuracy measured as MAE between projected predicted RTs and experimental RTs for the four reference CMs (FEM long, LIFE old, FEM orbitrap plasma, RIKEN) should remain competitive using only ~10 identified molecules.
- Meta-training convergence: residual variance estimated by type-II maximum likelihood over 250 Adam epochs at learning rate 0.01 should stabilize.
- Scalars learned from predicted RTs: inputs robustly scaled to [0,1] and outcomes to [-1,1] with median and 0.741*IQR.
- The saved GP model must exist under the saved_models folder after `make train_projections`.

## Limitations

- Requires ~10 identified molecules (experimental-predicted RT pairs) in the target chromatographic method; cannot be used when no molecules have been identified.
- Non-retained molecules (RT < 5 min) are excluded from the meta-tasks, so projections for non-retained compounds are out of scope.
- The polynomial kernel of degree 4 and constant mean were selected for this PredRet setting; other kernels may underperform, and the GP must be re-meta-trained (e.g. `make train_projections`) for new methods.
- Validated only on the four PredRet reference CMs (FEM long, LIFE old, FEM orbitrap plasma, RIKEN); README notes meta-training in smoke tests proceeds for only a few epochs, so full training requires removing the smoke-test flag.

## Evidence

- [methods] Meta-learning of GP prior over projection functions: "meta-tasks are used to learn a useful prior distribution, which is then used as starting point to solve the target-task."
- [intro] Meta-learning provides credible intervals, not just point estimates: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT) projections between CMs, and not only a point"
- [abstract] Projection learns from as few as 10 identified molecules: [evidence span withheld — anchor status: not_found]
- [readme] README: meta-train a GP for projections using the PredRet database: "make train_projections`: Meta-trains a Gaussian Process (GP) for computing projections between CMs using the PredRet database for a few epochs. The resulting GP is stored in the `saved_models` folder."
- [readme] README: branch paper is reference for reproducibility: "branch `paper` should be used as reference for reproducing the results of the paper."
