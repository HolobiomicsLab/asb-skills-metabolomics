---
name: accurate-mass-candidate-search
description: Use when an LC-MS feature has an accurate m/z and an experimental retention
  time, and you need to enumerate putative metabolite candidates and rank them using
  projected predicted RTs.
when_to_use_negative:
- The feature has no experimentally acquired RT (e.g., no chromatographic method to
  project onto), so ranking must rely on accurate mass alone
- You need spectral (MS/MS) matching rather than mass-plus-RT annotation — this skill
  only searches m/z against HMDB and ranks by predicted RT
- The target chromatographic method has fewer than ~50 identified molecules to fit
  the quartic RT projection regression
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment for the quartic least-squares RT projection regression (stats
    poly) and ROC/top-3 evaluation
- name: Keras for R
  role: Trains the deep learning model that produces the DLM-predicted RTs used in
    candidate ranking
- name: HMDB
  role: Source of the 6,823-molecule candidate list (molecules with KEGG numbers)
    searched at 10 ppm by accurate mass
- name: PredRet database
  role: Provides the experimental RT data for ECMs 1-4 (FEM_long, LIFE_old, RIKEN,
    FEM_orbitrap_plasma) used for projection and ranking
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:a1731eeb6a81bdb1b906c23f03417940dee7776fef29e339b0a18cf1e4e4771f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Keras for R
  - PredRet database
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# accurate-mass-candidate-search

## Summary

Retrieves all putative molecular identities for an unknown feature by accurate mass search of its protonated/deprotonated ion m/z (10 ppm tolerance) against the HMDB molecules with KEGG numbers, then ranks candidates by RT error so the correct identity lands in the top 3 in ~70% of cases.

## When to use

Use when an LC-MS feature has an accurate m/z and an experimental retention time, and you need to enumerate putative metabolite candidates and rank them using projected predicted RTs.

## When NOT to use

- The feature has no experimentally acquired RT (e.g., no chromatographic method to project onto), so ranking must rely on accurate mass alone
- You need spectral (MS/MS) matching rather than mass-plus-RT annotation — this skill only searches m/z against HMDB and ranks by predicted RT
- The target chromatographic method has fewer than ~50 identified molecules to fit the quartic RT projection regression

## Inputs

- List of unknown LC-MS compounds with accurate m/z (protonated/deprotonated ions) and experimental RTs from a PredRet ECM (FEM_long, LIFE_old, RIKEN, or FEM_orbitrap_plasma)
- HMDB molecule list (6,823 molecules with KEGG numbers)
- DLM-predicted retention times for the HMDB molecules
- Random subset of 50 identified molecules per ECM to calibrate the RT projection

## Outputs

- Per-compound list of putative candidate identities retrieved at 10 ppm mass tolerance
- Ranked candidates per compound ordered by RT error between projected predicted RT and experimental RT
- ROC curves and AUC per ECM (expected 0.65-0.69) with optimal RT-error filtering threshold
- Results table compiling projection errors, ROC AUC, optimal thresholds, and top-3 percentages per ECM

## How to apply

Load the HMDB molecule list (6,823 molecules with KEGG numbers) plus DLM-predicted RTs and PredRet ECM 1-4 experimental RT data (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). For each chromatographic method, build a predicted-to-experimental RT projection function via robust polynomial (quartic) least-squares regression fitted with the R stats poly function on a random subset of 50 identified molecules per ECM. For each compound, perform an accurate mass search at 10 ppm of the protonated/deprotonated ion m/z against the HMDB molecules with KEGG numbers to retrieve all putative candidates. Compute the RT error between the projected predicted RT and the experimental RT for each candidate, rank candidates by that error, and (optionally) generate ROC curves by discretizing relative RT error from 0 to 100% in 2.5% intervals to pick the filtering threshold maximizing TPR while minimizing FPR. The rationale is that accurate mass alone yields many putative candidates, but predicted RT narrows the correct identity to the top 3 in ~70% of cases (ROC AUC 0.65-0.69 across ECMs).

## Related tools

- **R** (Environment for the quartic least-squares RT projection regression (stats poly) and ROC/top-3 evaluation)
- **Keras for R** (Trains the deep learning model that produces the DLM-predicted RTs used in candidate ranking)
- **HMDB** (Source of the 6,823-molecule candidate list (molecules with KEGG numbers) searched at 10 ppm by accurate mass)
- **PredRet database** (Provides the experimental RT data for ECMs 1-4 (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma) used for projection and ranking)

## Examples

```
Rscript -e 'fit <- lm(exp_RT ~ poly(pred_RT, degree = 4, raw = TRUE), data = calib50); preds <- predict(fit, newdata = ecm); rank(candidates, by = abs(preds - exp_RT))'
```

## Evaluation signals

- Predicted RT ranks the correct identity among the top three candidates in ~70% of cases for all ECMs 1-4
- ROC AUC values fall within 0.65-0.69 for each ECM, matching the paper's Fig. 5d, i, n, s
- Accurate mass search uses a 10 ppm tolerance and returns candidates from HMDB molecules that have KEGG numbers
- RT projection regression is quartic polynomial least squares fitted on a random subset of 50 identified molecules per ECM
- ROC curves discretize relative RT error from 0 to 100% in 2.5% intervals, with the optimal threshold maximizing TPR while minimizing FPR

## Limitations

- Performance is bounded by ROC AUC 0.65-0.69; RT filtering alone cannot uniquely identify compounds and leaves ~30% of correct identities outside the top 3
- Depends on the availability of experimental RT data for the target chromatographic method to fit the projection function
- Restricted to HMDB molecules with KEGG numbers and reverse-phase chromatography data (SMRT dataset scope)

## Evidence

- [abstract] Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules.: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [abstract] the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time: "the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [methods] perform an accurate mass search (10 ppm) of the protonated/deprotonated ion m/z against the HMDB molecules with KEGG numbers to retrieve all putative candidates: [evidence span withheld — anchor status: not_found]
- [full_text] Generate ROC curves for each ECM by discretizing the relative RT error from 0 to 100% in 2.5% intervals: [evidence span withheld — anchor status: not_found]
