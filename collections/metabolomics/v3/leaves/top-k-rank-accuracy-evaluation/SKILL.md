---
name: top-k-rank-accuracy-evaluation
description: Use when a metabolomics annotation workflow must be scored by how often
  the true compound appears in the top-k ranked candidates, where ranking combines
  accurate mass search (e.g., 10 ppm m/z tolerance) with predicted RT error.
when_to_use_negative:
- No experimental RTs exist for the target chromatographic method, so no quartic projection
  model can be fitted on ~50 identified molecules
- Candidate filtering relies on MS/MS spectral matching or other orthogonal evidence
  rather than accurate mass plus predicted RT ranking
- The evaluation goal is standalone RT prediction performance (e.g., RMSE against
  measured RT) rather than candidate ranking/filtering accuracy
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R (stats::poly)
  role: Fits the robust quartic polynomial least-squares regression used to project
    DLM-predicted RTs onto each experimental chromatographic method
- name: Keras for R
  role: Provides the deep learning model that generates predicted retention times
    for the candidate molecules
- name: HMDB
  role: Source of the 6,823-molecule reference list with KEGG numbers used for accurate
    mass search candidate retrieval
- name: PredRet database
  role: Supplies experimental RTs for ECMs 1-4 (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
    against which projected RTs are compared
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:4afeb547a3c9b40e9ef76dff56930bccf33a5e10c6ff37b42907691926821cf4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Keras for R
  - PredRet database
  - R
  - R (stats::poly)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# top-k-rank-accuracy-evaluation

## Summary

Evaluates whether predicted retention times from a deep learning model (trained on the METLIN SMRT dataset) can rank the correct molecular identity among the top 3 candidates retrieved by accurate mass search, across experimental chromatographic methods (PredRet ECMs 1-4).

## When to use

Use when a metabolomics annotation workflow must be scored by how often the true compound appears in the top-k ranked candidates, where ranking combines accurate mass search (e.g., 10 ppm m/z tolerance) with predicted RT error.

## When NOT to use

- No experimental RTs exist for the target chromatographic method, so no quartic projection model can be fitted on ~50 identified molecules
- Candidate filtering relies on MS/MS spectral matching or other orthogonal evidence rather than accurate mass plus predicted RT ranking
- The evaluation goal is standalone RT prediction performance (e.g., RMSE against measured RT) rather than candidate ranking/filtering accuracy

## Inputs

- Reference molecule list with KEGG numbers (e.g., 6,823 HMDB molecules)
- DLM-predicted retention times (Keras deep learning model trained on METLIN SMRT data)
- Experimental RTs from PredRet ECM 1-4 datasets (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Accurate m/z values of protonated/deprotonated ions from the test compounds

## Outputs

- Top-3 candidate ranking accuracy per experimental chromatographic method (expect ~70%)
- ROC curves and AUC values per ECM (expect 0.65-0.69) with optimal relative RT error thresholds
- Per-candidate RT errors and ranked putative identity lists
- Compiled results table of projection errors, ROC AUC, optimal thresholds, and top-3 percentages

## How to apply

Load a reference molecule list (e.g., 6,823 HMDB molecules with KEGG numbers), DLM-predicted RTs, and experimental RTs from PredRet ECMs 1-4 (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). For each ECM, build a predicted-to-experimental RT projection by fitting a robust quartic polynomial least-squares regression (R stats::poly) on a random subset of 50 identified molecules. For each compound, perform an accurate mass search at 10 ppm of the protonated/deprotonated ion m/z against the reference list to retrieve all putative candidates, then rank candidates by the absolute RT error between projected predicted RT and experimental RT. Report top-3 ranking accuracy per ECM and, optionally, ROC curves computed by discretizing relative RT error from 0 to 100% in 2.5% intervals to find the threshold maximizing TPR while minimizing FPR.

## Related tools

- **R (stats::poly)** (Fits the robust quartic polynomial least-squares regression used to project DLM-predicted RTs onto each experimental chromatographic method)
- **Keras for R** (Provides the deep learning model that generates predicted retention times for the candidate molecules)
- **HMDB** (Source of the 6,823-molecule reference list with KEGG numbers used for accurate mass search candidate retrieval)
- **PredRet database** (Supplies experimental RTs for ECMs 1-4 (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma) against which projected RTs are compared)

## Examples

```
proj <- lm(exp_rt ~ poly(pred_rt, 4), data = ecm_train_50); ranked <- candidates[order(abs(predict(proj, candidates) - candidates$exp_rt)), ]
```

## Evaluation signals

- Top-3 ranking accuracy is approximately 70% for every ECM tested (ECMs 1-4), matching the paper's reported figures (Fig. 5e, j, o, t)
- ROC AUC values fall within the 0.65-0.69 range for all ECMs (Fig. 5d, i, n, s)
- The projection model is a quartic (degree-4) polynomial fitted on exactly 50 randomly selected identified molecules per ECM via R stats::poly
- Accurate mass search retrieves candidates at 10 ppm tolerance of the protonated/deprotonated ion m/z before RT-based ranking
- ROC curves are computed by discretizing relative RT error from 0 to 100% in 2.5% intervals

## Limitations

- The ~70% top-3 accuracy and 0.65-0.69 AUC apply to the specific PredRet ECMs 1-4 and reverse-phase chromatography; performance depends on fitting a projection model per new chromatographic method
- Ranking is limited to candidates retrieved by a 10 ppm accurate mass search, so incorrect masses outside tolerance are never considered
- Requires a random subset of 50 identified molecules per ECM to train the projection function; methods with few identified compounds cannot be projected

## Evidence

- [intro] Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
- [abstract] SMRT dataset description: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [intro] Model projection scope: "We analyzed the model's predictive ability along with the ability to project predicted RT onto other CMs"
- [results] Top-3 finding across ECMs: [evidence span withheld — anchor status: not_found]
