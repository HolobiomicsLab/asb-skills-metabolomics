---
name: gp-projection-uncertainty-scoring
description: Use when annotating metabolites with only m/z and retention time (no
  fragmentation spectra) and you need to rank mass-filtered candidates by RT plausibility
  with uncertainty.
when_to_use_negative:
- Fragmentation (MS/MS) spectra are available — spectral matching provides stronger
  evidence than RT-based z-score ranking
- The unknown was not retained on the reversed-phase column (RT < 5 min) and the target
  method lacks a non-retained class, since the RT prediction model treats non-retained
  molecules separately
- Fewer than ~10 identified molecules exist for the target chromatographic method,
  so the meta-learned GP projection cannot be reliably trained
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: cmmrt
  role: Provides the DNN RT predictor, meta-learned GP projection code, and the meta-training/evaluation
    routines used to compute z-scores and rank candidates
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the 5,666 descriptors and 2,214 fingerprints (MACCS166, ECFP, PFP)
    that are the input features for the DNN RT predictions being projected
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:73a167d1c1ca45b72f8979e4f6c113df4095b9618c3840d2b1e31fd35e5ddd46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gp-projection-uncertainty-scoring

## Summary

Ranks mass-filtered metabolite candidates by the GP-projection RT z-score z = |y* - mu(x*)| / sigma(x*), where mu and sigma come from a meta-learned Gaussian Process projecting DNN-predicted RTs to a target chromatographic method; the correct molecule landed in the top three in ~68% of cases.

## When to use

Use when annotating metabolites with only m/z and retention time (no fragmentation spectra) and you need to rank mass-filtered candidates by RT plausibility with uncertainty.

## When NOT to use

- Fragmentation (MS/MS) spectra are available — spectral matching provides stronger evidence than RT-based z-score ranking
- The unknown was not retained on the reversed-phase column (RT < 5 min) and the target method lacks a non-retained class, since the RT prediction model treats non-retained molecules separately
- Fewer than ~10 identified molecules exist for the target chromatographic method, so the meta-learned GP projection cannot be reliably trained

## Inputs

- SMILES/mol/SDF structures of candidate molecules (e.g., HMDB entries with KEGG numbers) for RT prediction
- alvaDesc-generated fingerprints and descriptors (MACCS166, ECFP, path fingerprints) for each candidate
- Experimental RTs of ~10-50 identified molecules in the target chromatographic method (meta-training pairs)
- PredRet database experimental RTs for meta-training the GP prior
- Measured accurate masses (m/z) of unknown features, queried with 10 ppm tolerance
- Measured RT of the unknown feature in the target chromatographic method

## Outputs

- GP posterior mean mu(x*) and standard deviation sigma(x*) of projected RT per candidate molecule
- RT z-score z = |y* - mu(x*)| / sigma(x*) per candidate
- Ranked candidate list per mass query (top-3 shortlist)
- Top-three annotation accuracy per chromatographic method and training-set size (e.g., annotation_ranking_accuracy.csv with standard errors and mass-only baselines)

## How to apply

Build an RT database by predicting retention times for candidate molecules (e.g., the 6,823 KEGG-numbered HMDB molecules) with a DNN trained on the 80,038-experiment SMRT dataset, whose features (5,666 descriptors plus MACCS166, ECFP, and path fingerprints) were generated with alvaDesc. Meta-train a GP projection on leave-one-CM-out PredRet tasks (constant mean, degree-4 polynomial kernel) using the few identified molecules of the target chromatographic method, then project each predicted RT to the target method, obtaining posterior mean mu(x*) and standard deviation sigma(x*). For each unknown, simulate/perform an accurate-mass search with 10 ppm tolerance, add Gaussian mass noise (zero mean, sigma = 10/3 ppm, truncated to ±10 ppm), and rank returned candidates by z = |y* - mu(x*)| / sigma(x*). Restrict evaluation to queries returning more than three candidates, compute top-3 accuracy, and compare against a mass-only random-tie baseline. Rationale: the meta-learned GP generalizes from as few as 10 identified molecules while still yielding calibrated confidence intervals, so small absolute RT errors translate into useful z-score rankings.

## Related tools

- **cmmrt** (Provides the DNN RT predictor, meta-learned GP projection code, and the meta-training/evaluation routines used to compute z-scores and rank candidates) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the 5,666 descriptors and 2,214 fingerprints (MACCS166, ECFP, PFP) that are the input features for the DNN RT predictions being projected)

## Examples

```
make test_projections  # or: python cmmrt/rt/projections_to_different_cm.ipynb; rank by z = |y* - mu| / sigma
```

## Evaluation signals

- Global mean top-three accuracy reaches approximately 68% with 10 training points and approximately 70% with 50 training points, consistent with Table 5
- Accuracy exceeds the mass-only random-tie baseline (e.g., 59.57% for FEM long)
- Projection MAE on target CMs (FEM long, FEM orbitrap plasma, LIFE old, RIKEN) is competitive when meta-trained with as few as 10 molecules
- Mass search with truncated Gaussian noise (sigma = 10/3 ppm, ±10 ppm) always returns the true molecule among candidates for queries with >3 candidates
- GP posterior sigma(x*) is non-degenerate and z-scores are computed from the leave-one-CM-out meta-trained projection, not a naive fit

## Limitations

- Ranking relies on RT plausibility only; when two candidates share similar m/z and similar projected RTs, z-scores cannot discriminate them
- Performance depends on the quality of the DNN RT prediction (MAE 39.2±1.2 s, median 17.2±0.9 s on SMRT) and on the representativeness of PredRet CMs used for meta-training
- Non-retained molecules (RT < 5 min) are treated as a classification problem in the DNN, so RT-based ranking is less informative for unretained features
- The 10 ppm mass-search tolerance and Gaussian noise truncation are simulation choices; real mass accuracies may differ

## Evidence

- [abstract] correct molecule among top three candidates filtered by mass and ranked by z-scores: "our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
- [intro] confidence intervals from meta-learned GP: "This approach has the advantage of being able to generalize from a small training set while providing confidence intervals for the Retention Time (RT) projections between CMs, and not only a point"
- [methods] z-score formula from workflow card: [evidence span withheld — anchor status: not_found]
- [abstract] MAE of DNN RT predictor: "achieving a mean and median absolute errors of 39.2±1.2 s and 17.2 ± 0.9 s, respectively"
- [abstract] few identified molecules suffice for meta-training: [evidence span withheld — anchor status: not_found]
