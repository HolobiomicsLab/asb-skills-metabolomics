---
name: retention-time-projection-regression
description: Use when you have DLM-predicted RTs, a molecule library with accurate
  masses (e.g. HMDB), and experimental RTs from a target chromatographic method, and
  you need to rank/filter candidate identities from 10 ppm accurate-mass search results.
when_to_use_negative:
- The target chromatographic method is not reverse-phase LC (the SMRT dataset and
  DLM predictions cover experimentally acquired reverse-phase RTs only).
- Experimental RTs are already available for the query compounds in the target method,
  making projected predicted RTs unnecessary.
- You need identity confirmation rather than candidate ranking — an RT-error rank
  alone cannot uniquely identify a molecule when mass-search candidates share similar
  RTs.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: R
  role: Fits the quartic polynomial least-squares projection (stats::poly within lm)
    between predicted and experimental RTs for each ECM
- name: Keras for R
  role: Trains the deep learning model (DLM) that produces the predicted RTs that
    are projected onto each chromatographic method
- name: HMDB
  role: Source of molecule identities, KEGG numbers, and accurate masses used as the
    candidate space for 10 ppm accurate-mass search
- name: PredRet database
  role: 'Provides experimental RT datasets (ECMs 1-4: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
    used to fit and validate each projection function'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:93ca90bd334abe76fb3f16c60845292075408952ee1c5606b78f85c919602c1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Keras for R
  - PredRet database
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# retention-time-projection-regression

## Summary

Projects deep-learning-predicted reverse-phase retention times (RTs) onto other chromatographic methods via quartic polynomial regression, then uses the projected RT plus accurate-mass search to rank candidate molecular identities from real metabolomics experiments.

## When to use

Use when you have DLM-predicted RTs, a molecule library with accurate masses (e.g. HMDB), and experimental RTs from a target chromatographic method, and you need to rank/filter candidate identities from 10 ppm accurate-mass search results.

## When NOT to use

- The target chromatographic method is not reverse-phase LC (the SMRT dataset and DLM predictions cover experimentally acquired reverse-phase RTs only).
- Experimental RTs are already available for the query compounds in the target method, making projected predicted RTs unnecessary.
- You need identity confirmation rather than candidate ranking — an RT-error rank alone cannot uniquely identify a molecule when mass-search candidates share similar RTs.

## Inputs

- HMDB molecule list with KEGG numbers and monoisotopic masses (e.g. 6,823 molecules)
- Deep-learning-model (DLM/Keras) predicted retention times per molecule
- Experimental RT data for the target chromatographic method (e.g. PredRet ECM datasets: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Accurate-mass m/z of the protonated/deprotonated ion per measured feature

## Outputs

- Per-ECM quartic polynomial projection function (predicted-to-experimental RT mapping)
- Ranked list of putative candidate identities per feature, ordered by RT error
- ROC curves with AUC values (expected 0.65–0.69) and optimal RT-error filtering thresholds (relative error discretized 0–100% in 2.5% bins)
- Top-3 ranking accuracy per ECM, compiled with projection errors, ROC AUC, and thresholds into a single results table

## How to apply

Load the molecule list (e.g. 6,823 HMDB molecules with KEGG numbers), the DLM-predicted RTs, and experimental RT data for the target ECM (e.g. PredRet FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). Build the predicted-experimental projection function for each ECM by fitting a robust quartic (4th-degree) polynomial least-squares regression with R's stats::poly on a random subset of 50 identified molecules per ECM. For each compound, perform an accurate mass search at 10 ppm tolerance of the protonated/deprotonated ion m/z against the HMDB entries to retrieve all putative candidates. Compute the RT error between the projected predicted RT and the experimental RT for each candidate and rank putative identities by that error. Optionally build ROC curves by discretizing relative RT error from 0 to 100% in 2.5% intervals to choose a filtering threshold; expected performance is correct identity in the top 3 for ~70% of cases with ROC AUC 0.65–0.69.

## Related tools

- **R** (Fits the quartic polynomial least-squares projection (stats::poly within lm) between predicted and experimental RTs for each ECM)
- **Keras for R** (Trains the deep learning model (DLM) that produces the predicted RTs that are projected onto each chromatographic method)
- **HMDB** (Source of molecule identities, KEGG numbers, and accurate masses used as the candidate space for 10 ppm accurate-mass search)
- **PredRet database** (Provides experimental RT datasets (ECMs 1-4: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma) used to fit and validate each projection function)

## Examples

```
proj <- lm(rt_exp ~ poly(rt_pred, 4), data = calib_50_mols); candidates$rt_err <- abs(predict(proj, candidates) - candidates$rt_exp)
```

## Evaluation signals

- Projection regression is fitted on exactly 50 randomly selected identified molecules per ECM with a quartic (degree-4) poly model.
- Correct molecular identity appears among the top 3 RT-error-ranked candidates for approximately 70% of cases across all ECMs tested.
- ROC AUC values from the RT-error discretization fall within 0.65–0.69 (matching Fig. 5d, i, n, s).
- Accurate-mass search retrieves candidates at a 10 ppm tolerance of the protonated/deprotonated ion m/z.
- ROC discretization uses 2.5% relative-RT-error bins spanning 0–100%, yielding thresholds that maximize TPR while minimizing FPR.

## Limitations

- Performance depends on the projection regression quality: the quartic fit is learned from only 50 identified molecules per ECM, so poorly represented chromatographic methods may project inaccurately.
- Ranking accuracy is bounded (~70% top-3, AUC 0.65–0.69); correct identity is not always the top-ranked candidate and RT-error filtering cannot eliminate all false positives.
- The underlying DLM RT predictions and SMRT dataset cover reverse-phase chromatography, so transferability to other separation modes is untested here.

## Evidence

- [abstract] top-3 ranking by projected RT error: "the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [intro] skill motivation: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
- [intro] projection step: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
