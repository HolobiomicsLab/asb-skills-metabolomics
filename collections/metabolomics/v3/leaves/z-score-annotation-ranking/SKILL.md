---
name: z-score-annotation-ranking
description: Use when annotating metabolites in untargeted LCMS studies where fragmentation
  spectra are unavailable and annotation must rely only on m/z ratios and retention
  times, i.e., when mass-filtered candidate lists must be ranked by RT agreement.
when_to_use_negative:
- When MS/MS fragmentation spectra are available — spectral matching is more informative
  and this skill is designed for the m/z + RT-only regime
- When no identified molecules from the target chromatographic method exist to fit
  the projection — the GP needs as few as 10 known experimental-predicted RT pairs
- For GC-based or non-reversed-phase LCMS methods outside the reversed-phase LCMS
  setting of SMRT/PredRet used to train the predictor and meta-learned prior
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: cmmrt
  role: Python package implementing the DNN RT predictor, GP projection, and metabolite
    annotation workflow; branch 'paper' reproduces the article results
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the ECFP, MACCSFP and PFP fingerprints (plus 5,666 descriptors)
    used to build the predicted-RT database that is queried after z-score ranking
- name: CEU Mass Mediator
  role: Metabolite annotation platform into which the proposed projection and ranking
    workflow is intended to be integrated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:147fd5501b6210c903f20d0e3f9ed31de9c0a1017dec59faaed2b6307251562a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - CEU Mass Mediator
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# z-score-annotation-ranking

## Summary

A probabilistic metabolite annotation ranking step in which candidates retrieved by accurate-mass filtering are scored by how well their experimental RT agrees with a projected RT prediction, using the z-score z = |y_exp - mu(x_pred)| / sigma(x_pred) from a GP posterior.

## When to use

Use when annotating metabolites in untargeted LCMS studies where fragmentation spectra are unavailable and annotation must rely only on m/z ratios and retention times, i.e., when mass-filtered candidate lists must be ranked by RT agreement.

## When NOT to use

- When MS/MS fragmentation spectra are available — spectral matching is more informative and this skill is designed for the m/z + RT-only regime
- When no identified molecules from the target chromatographic method exist to fit the projection — the GP needs as few as 10 known experimental-predicted RT pairs
- For GC-based or non-reversed-phase LCMS methods outside the reversed-phase LCMS setting of SMRT/PredRet used to train the predictor and meta-learned prior

## Inputs

- Experimental retention times of unknown queries from a target chromatographic method (e.g., FEM long, FEM orbitrap plasma, LIFE old, RIKEN PredRet CMs)
- Experimental RTs of ~10-50 known/annotated molecules from the same chromatographic method, for fitting the GP projection
- Predicted-RT database of HMDB molecules with KEGG numbers (fingerprint features from alvaDesc: ECFP, MACCSFP, PFP)
- Accurate precursor m/z values for the queries

## Outputs

- Per-query ranked list of candidate annotations scored by z = |y_exp - mu(x_pred)| / sigma(x_pred)
- Top-3 candidate sets per query and the overall percentage of queries where the true molecule is in the top three
- Mass-only random-tie-break baseline comparison

## How to apply

First build a predicted-RT database: generate ECFP, MACCSFP and PFP fingerprints with alvaDesc for the target library (6,823 molecules with KEGG numbers in HMDB) and predict RTs with the DNN trained on the 80,038 SMRT retention times. Then learn a projection from predicted to experimental RTs using a GP with a meta-learned prior fitted on a few known-molecule experimental-predicted pairs, which yields both a posterior mean mu(x_pred) and a posterior standard deviation sigma(x_pred). Filter molecules by m/z (accurate mass search at 10 ppm tolerance in the paper's evaluation), project the predicted RTs of surviving candidates, and query the projected database with each experimental RT. For every candidate compute z = |y_exp - mu(x_pred)| / sigma(x_pred), rank candidates by z-score, and report the percentage of queries where the true molecule is among the top three, against a mass-only random-tie-break baseline.

## Related tools

- **cmmrt** (Python package implementing the DNN RT predictor, GP projection, and metabolite annotation workflow; branch 'paper' reproduces the article results) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the ECFP, MACCSFP and PFP fingerprints (plus 5,666 descriptors) used to build the predicted-RT database that is queried after z-score ranking)
- **CEU Mass Mediator** (Metabolite annotation platform into which the proposed projection and ranking workflow is intended to be integrated)

## Examples

```
make test_projections  # tests GP meta-trained projections on FEM long, LIFE old, FEM orbitrap plasma, RIKEN; results under results/projection
```

## Evaluation signals

- Top-3 recovery: the true molecule should appear among the top three z-score-ranked candidates in about 68% of queries in the paper's evaluation
- Ranking should outperform the mass-only random-tie-break baseline on the same filtered candidate sets
- Only queries returning more than three candidates after the 10 ppm accurate-mass search should be scored
- z-scores must be computed from both the GP posterior mean and the posterior standard deviation, not from point RT predictions alone
- Projection must be fitted with the meta-learned GP prior frozen (residual variance only), with leave-one-CM-out meta-training so target-CM data never enters meta-training

## Limitations

- The evaluation used simulated mass noise N(0, (10/3 ppm)^2) truncated to +/-10 ppm rather than fully real query data
- Queries returning three or fewer candidates after mass filtering are excluded, so performance is conditional on non-trivial candidate lists
- The GP prior must stay frozen during target fitting (only residual variance is fit with Adam, lr 0.01, 250 epochs) to preserve meta-learned generalization from few samples
- Applicable to reversed-phase LCMS; both retained and non-retained molecules (RT < 5 min treated as non-retained) were considered when training the underlying predictor

## Evidence

- [abstract] Top-3 recovery headline result: "our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
- [intro] Workflow steps 6-7: querying the projected database with experimental RTs: "The results retrieved from it would enable scoring candidates with similar m/z but different RTs (step 7)."
- [intro] Mass filtering precedes RT scoring: "The m/z ratios are used as a first filter to obtain candidate annotations from the database."
- [intro] Why z-scores (probabilistic) rather than point-estimate projections: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT)"
- [readme] Concrete implementation of projection fitting with the cmmrt notebooks: "map experimental retention times to the retention times predicted with a DNN (or viceversa). This is done by training a meta-learned GP prior on a small subset of known molecules."
