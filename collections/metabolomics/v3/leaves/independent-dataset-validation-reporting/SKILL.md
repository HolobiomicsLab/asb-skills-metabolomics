---
name: independent-dataset-validation-reporting
description: Use when a new feature-clustering method must be validated against established
  tools (e.g. CAMERA) on a dataset independent of the one used to develop the method,
  with clustering-quality metrics reported in a reproducible table.
when_to_use_negative:
- The clustering method is being evaluated on the same dataset used for its development
  or parameter tuning — an independent dataset is required for unbiased validation
- The analysis goal is to perform or tune the feature clustering itself (e.g. setting
  ramclustR st/maxt parameters) rather than to benchmark and report comparative performance
- The dataset lacks idMS/MS (indiscriminant tandem MS) data, so MS/MS spectral similarity
  — the primary validation metric here — cannot be computed
edam_operation: http://edamontology.org/operation_2428
edam_topics: []
tools:
- name: RAMClustR
  role: Method under test; unsupervised post-XCMS feature clustering evaluated on
    the independent urine dataset
  repo: https://github.com/cbroeckl/RAMClustR
- name: CAMERA
  role: Comparison baseline; its groupFWHM, groupCorr, and groupDen grouping functions
    provide the reference clustering results
- name: R
  role: Analysis environment for constructing the benchmark data frame and writing
    table1_urine_block.csv
- name: XCMS
  role: Upstream feature detection producing the aligned features that all clustering
    methods are applied to
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:de44a1118e4cd02bc2a91cdc40d1753489923d4c2fef34e4cbc45d0a13c53242
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - CAMERA
  - R
  - RAMClustR
  - XCMS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# independent-dataset-validation-reporting

## Summary

Benchmarking skill for validating feature-clustering performance (RAMClustR vs CAMERA's groupFWHM/groupCorr/groupDen) on an independent metabolomics dataset, reporting MS/MS spectral similarity, multi-feature cluster counts, and singleton percentages as a comparison table.

## When to use

Use when a new feature-clustering method must be validated against established tools (e.g. CAMERA) on a dataset independent of the one used to develop the method, with clustering-quality metrics reported in a reproducible table.

## When NOT to use

- The clustering method is being evaluated on the same dataset used for its development or parameter tuning — an independent dataset is required for unbiased validation
- The analysis goal is to perform or tune the feature clustering itself (e.g. setting ramclustR st/maxt parameters) rather than to benchmark and report comparative performance
- The dataset lacks idMS/MS (indiscriminant tandem MS) data, so MS/MS spectral similarity — the primary validation metric here — cannot be computed

## Inputs

- Independent metabolomics dataset (e.g. urine LC-MS/idMS/MS data) processed with XCMS into aligned features
- Clustering outputs from each method compared: CAMERA groupFWHM, groupCorr, groupDen, and RAMClustR
- Previously reported benchmark values (Table 1 urine block) with metrics MSMS similarity, n Clus (>1), and Sing (%)

## Outputs

- table1_urine_block.csv — one row per clustering method (groupFWHM, groupCorr, groupDen, ramclustR) with columns MSMS similarity (0.106/0.059/0.020/0.228), n Clus (>1) (290/332/35/827), and Sing (4.88%/61.77%/0.00%/32.75%)
- A verified benchmark comparison demonstrating RAMClustR's higher MS/MS similarity (0.228), more multi-feature clusters (827), and different singleton rates versus CAMERA methods

## How to apply

Run each clustering method (groupFWHM, groupCorr, groupDen from CAMERA, and ramclustR) on the same independently acquired dataset — here a urine metabolomics dataset processed with XCMS feature detection — rather than the dataset used for method development. Score each method on three metrics: MS/MS spectral similarity (assessing whether clustered features form coherent idMS/MS spectra), the number of multi-feature clusters (n Clus >1), and the percentage of ungrouped singletons (Sing). Load the reported Table 1 urine-block values into an R data frame with one row per method — MSMS similarity 0.106/0.059/0.020/0.228, n Clus (>1) 290/332/35/827, Sing 4.88%/61.77%/0.00%/32.75% — write it to table1_urine_block.csv, and verify each cell against the reported value. The rationale is that features derived from the same compound share approximately the same retention time and quantitative trend across samples, so higher MS/MS similarity and more multi-feature clusters with fewer singletons indicate better deconvolution of compound-level spectra.

## Related tools

- **RAMClustR** (Method under test; unsupervised post-XCMS feature clustering evaluated on the independent urine dataset) — https://github.com/cbroeckl/RAMClustR
- **CAMERA** (Comparison baseline; its groupFWHM, groupCorr, and groupDen grouping functions provide the reference clustering results)
- **R** (Analysis environment for constructing the benchmark data frame and writing table1_urine_block.csv)
- **XCMS** (Upstream feature detection producing the aligned features that all clustering methods are applied to)

## Examples

```
df <- data.frame(method=c("groupFWHM","groupCorr","groupDen","ramclustR"), msms_sim=c(0.106,0.059,0.020,0.228), n_clus_gt1=c(290,332,35,827), sing_pct=c(4.88,61.77,0.00,32.75)); write.csv(df, "table1_urine_block.csv", row.names=FALSE)
```

## Evaluation signals

- Cell-by-cell verification: every value in table1_urine_block.csv matches the reported Table 1 urine block (MSMS 0.106/0.059/0.020/0.228; n Clus (>1) 290/332/35/827; Sing 4.88%/61.77%/0.00%/32.75%)
- Direction-of-effect check: RAMClustR achieves the highest MS/MS similarity (0.228) and the most multi-feature clusters (827) among the compared methods
- Schema check: the CSV has exactly 4 rows (one per method: groupFWHM, groupCorr, groupDen, ramclustR) and 3 metric columns
- Range sanity: singleton percentages fall in 0–100 and MS/MS similarity values are non-negative and consistent with the reported magnitudes (0.020–0.228)

## Limitations

- MS/MS spectral similarity is only computable when the dataset includes idMS/MS data; the RAMClustR approach implicitly incorporates indiscriminant MS/MS by performing feature detection on both MS and idMS/MS data
- Singleton percentages (e.g. 0.00% for groupDen vs 32.75% for ramclustR on the urine dataset) are dataset-dependent and must be re-measured on each independent dataset rather than transferred
- Benchmarking requires identical upstream XCMS feature detection for all clustering methods, otherwise the comparison of cluster counts and singletons is confounded

## Evidence
