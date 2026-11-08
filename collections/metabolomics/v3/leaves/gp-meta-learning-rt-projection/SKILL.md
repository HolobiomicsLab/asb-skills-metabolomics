---
name: gp-meta-learning-rt-projection
description: Use when annotating metabolites from untargeted LC-MS runs where fragmentation
  spectra are unavailable and only m/z ratios and retention times can be used to rank
  candidate molecules.
when_to_use_negative:
- Fragmentation spectra (MS/MS) are available for the features, since spectral matching
  is stronger evidence than RT-based ranking
- The target chromatographic method has fewer than ~10 identified molecules, so there
  are too few experimental-predicted RT pairs to fit the residual variance
- The goal is RT prediction on the SMRT reversed-phase LC-MS setup itself rather than
  projection between chromatographic methods
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: cmmrt
  role: Python package implementing GP meta-training for RT projections and the DNN
    RT predictor; use the 'paper' branch to reproduce results
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Runtime environment for the cmmrt package and experiment scripts
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the ECFP, MACCSFP and PFP fingerprints used by the DNN whose predicted
    RTs are the source side of the projection
- name: RDKit
  role: Alternative fingerprint backend for training the DNN (train_with_rdkit.ipynb
    notebook)
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:bb6dd644abaec6fae6d64d3adf3a98943116260ffc42da5a722db5d913a24f5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gp-meta-learning-rt-projection

## Summary

A Bayesian meta-learning approach that learns a Gaussian Process prior over projection functions mapping DNN-predicted retention times to the experimental RTs of a specific chromatographic method, using as few as 10 identified molecules and yielding predictive standard deviations usable as z-score confidence measures for metabolite annotation.

## When to use

Use when annotating metabolites from untargeted LC-MS runs where fragmentation spectra are unavailable and only m/z ratios and retention times can be used to rank candidate molecules.

## When NOT to use

- Fragmentation spectra (MS/MS) are available for the features, since spectral matching is stronger evidence than RT-based ranking
- The target chromatographic method has fewer than ~10 identified molecules, so there are too few experimental-predicted RT pairs to fit the residual variance
- The goal is RT prediction on the SMRT reversed-phase LC-MS setup itself rather than projection between chromatographic methods

## Inputs

- Pairs of experimental RTs and DNN-predicted RTs for a few (10-50) known molecules in the target chromatographic method
- Meta-training set of chromatographic methods from the PredRet database
- Predicted-RT database of 6,823 HMDB molecules with KEGG numbers (ECFP, MACCSFP, PFP fingerprints from alvaDesc fed to the SMRT-trained DNN)

## Outputs

- A fitted meta-learned GP projection model (posterior mean mu and standard deviation sigma over RT projections)
- Projected-RT database mapping predicted RTs to the target method's experimental RT scale
- Z-scores per candidate molecule for ranking annotations

## How to apply

Preprocess RTs with log(1+x) followed by robust median/IQR scaling (factor 0.741) into the [0,1]/[-1,1] ranges required by the GP software. Meta-train a GP prior over projection functions (constant mean, polynomial kernel of degree 4) on PredRet chromatographic methods by minimizing the negative LOO log predictive probability, using a leave-one-CM-out split so target-CM data never enters meta-training. For a target chromatographic method (e.g. FEM long, FEM orbitrap plasma, LIFE old, RIKEN), fit only the residual variance by type-II maximum likelihood (Adam, lr 0.01, 250 epochs) from stratified subsamples of 10-50 annotated molecules, keeping the meta-learned prior frozen. Compute the z-score z = |y_exp - mu(x_pred)| / sigma(x_pred) from the GP posterior mean and standard deviation for each candidate and rank candidates by z-score. The code is implemented in the cmmrt Python package (branch 'paper' reproduces the paper's results).

## Related tools

- **cmmrt** (Python package implementing GP meta-training for RT projections and the DNN RT predictor; use the 'paper' branch to reproduce results) — https://github.com/constantino-garcia/cmmrt
- **Python** (Runtime environment for the cmmrt package and experiment scripts) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the ECFP, MACCSFP and PFP fingerprints used by the DNN whose predicted RTs are the source side of the projection)
- **RDKit** (Alternative fingerprint backend for training the DNN (train_with_rdkit.ipynb notebook)) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make test_projections
```

## Evaluation signals

- Projection MAE between predicted and experimental RTs on the four PredRet test CMs (FEM long, FEM orbitrap plasma, LIFE old, RIKEN) should be competitive with previous projection approaches even at 10 training molecules
- In 68% of cases the correct molecule should be among the top three candidates filtered by mass and ranked by z-scores (10 ppm mass tolerance)
- Leave-one-CM-out check: the target chromatographic method's data must not appear in meta-training
- Reported projection outputs must include confidence intervals (posterior sigma), not only point estimates
- Meta-training uses leave-one-CM-out so results/ CSVs under results/projection from `make test_projections` should show per-CM performance summaries

## Limitations

- Requires at least ~10 identified molecules in the target chromatographic method to fit the residual variance; performance degrades with fewer pairs
- The GP prior is meta-trained on PredRet methods, so projections to chromatographic setups very different from PredRet may be less reliable
- Only 6,823 HMDB molecules with KEGG numbers form the predicted-RT database, limiting the annotation candidate space
- Simulated mass noise N(0, (10/3 ppm)^2) truncated to +/-10 ppm is applied during evaluation, so reported top-3 rates reflect that noise model

## Evidence

- [methods] Meta-learning of GP prior over projection functions: "meta-tasks are used to learn a useful prior distribution, which is then used as starting point to solve the target-task."
- [abstract] Few-shot projection capability: [evidence span withheld — anchor status: not_found]
- [abstract] Z-score ranking outcome: "our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
- [intro] Confidence intervals from GP posterior: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT) projections between CMs, and not only a point"
- [readme] Testing procedure on four reference CMs: "Four CMs of reference are used for the test: FEM long, LIFE old, FEM orbitrap plasma and RIKEN. For each system, the GP is meta-trained using all other CMs from the PredRet database"
- [readme] Use paper branch for reproduction: "branch `paper` should be used as reference for reproducing the results of the paper"
