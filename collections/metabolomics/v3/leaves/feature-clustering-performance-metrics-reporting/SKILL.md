---
name: feature-clustering-performance-metrics-reporting
description: Use when evaluating or benchmarking how well competing feature-grouping
  methods deconvolute MS/MS feature clusters from an XCMS feature table and you need
  to report MSMS similarity, cluster counts, and singleton percentages in a reproducible
  comparison table.
when_to_use_negative:
- When the task is feature detection or alignment itself (e.g. running XCMS on raw
  CDF files) rather than reporting/evaluating the performance of an already-chosen
  clustering method.
- When the dataset is not one for which published benchmark values exist — do not
  fabricate MSMS similarity, n Clus >1, or per Sing numbers for new datasets; this
  skill reproduces reported comparison rows.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RAMClustR
  role: The ramclustR clustering method being benchmarked against CAMERA grouping
    functions; its reported CSF-row metrics (0.382 MSMS similarity, 3248 clusters
    >1, 15.47% singletons) are tabulated.
  repo: https://github.com/cbroeckl/RAMClustR
- name: R
  role: Used to assemble the benchmark rows into a data frame and write the table
    to CSV (write.csv).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/ac501530d
    title: 'RAMClust: A Novel Feature Clustering Method Enables Spectral-Matching-Based
      Annotation for Metabolomics Data'
  source_content_hash: sha256:c0f2946e94027f7dfce991043f78dbf1755f654b27067b990cdf86d207608c56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cbroeckl/RAMClustR
  tools:
  - R
  - RAMClustR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# feature-clustering-performance-metrics-reporting

## Summary

Reports comparative performance metrics for post-XCMS feature clustering methods (RAMClustR, groupFWHM, groupCorr, groupDen) by assembling benchmark table rows such as Table 1's CSF block: MSMS similarity, number of clusters with two or more features (n Clus >1), and percentage of singletons (per Sing).

## When to use

Use when evaluating or benchmarking how well competing feature-grouping methods deconvolute MS/MS feature clusters from an XCMS feature table and you need to report MSMS similarity, cluster counts, and singleton percentages in a reproducible comparison table.

## When NOT to use

- When the task is feature detection or alignment itself (e.g. running XCMS on raw CDF files) rather than reporting/evaluating the performance of an already-chosen clustering method.
- When the dataset is not one for which published benchmark values exist — do not fabricate MSMS similarity, n Clus >1, or per Sing numbers for new datasets; this skill reproduces reported comparison rows.

## Inputs

- Reported values from the results-section Table 1 CSF block comparing ramclustR, groupFWHM, groupCorr, and groupDen
- MSMS similarity score per method (e.g. ramclustR 0.382, groupFWHM 0.202, groupCorr 0.177, groupDen 0.043)
- Counts of clusters with two or more features (n Clus >1) per method
- Percentage of singletons (per Sing) per method

## Outputs

- A 4-row data frame or CSV (e.g. table1_csf_rows.csv) with columns msms_similarity, n_clus_gt1, per_sing and one row per grouping method
- Benchmark table reproducing the reported CSF dataset rows

## How to apply

For each clustering method in the comparison (RAMClustR's ramclustR, plus CAMERA-style groupFWHM, groupCorr, and groupDen), record three metrics on the same feature-detected dataset: MSMS similarity (ramclustR 0.382, groupFWHM 0.202, groupCorr 0.177, groupDen 0.043 on the CSF dataset), the count of clusters containing two or more features (n Clus >1: 3248, 535, 784, 39 respectively), and the percentage of singletons (per Sing: 15.47%, 0.43%, 29.56%, 0.00%). In R, build a data frame with one row per method and columns msms_similarity, n_clus_gt1, and per_sing, then write the table with write.csv (e.g. table1_csf_rows.csv) so each cell matches the reported values exactly. RAMClustR's advantage — higher MSMS similarity driven by simultaneous use of MS and idMS/MS feature-feature relationships — is the rationale for including it alongside CAMERA grouping functions.

## Related tools

- **RAMClustR** (The ramclustR clustering method being benchmarked against CAMERA grouping functions; its reported CSF-row metrics (0.382 MSMS similarity, 3248 clusters >1, 15.47% singletons) are tabulated.) — https://github.com/cbroeckl/RAMClustR
- **R** (Used to assemble the benchmark rows into a data frame and write the table to CSV (write.csv).)

## Examples

```
write.csv(data.frame(method=c('groupFWHM','groupCorr','groupDen','ramclustR'), msms_similarity=c(0.202,0.177,0.043,0.382), n_clus_gt1=c(535,784,39,3248), per_sing=c(0.43,29.56,0.00,15.47)), 'table1_csf_rows.csv')
```

## Evaluation signals

- Output CSV contains exactly 4 data rows (plus header), one per method: groupFWHM, groupCorr, groupDen, ramclustR.
- Each cell exactly matches reported values, e.g. the ramclustR row equals 0.382, 3248, 15.47.
- groupDen row equals 0.043, 39, 0.00 and groupFWHM row equals 0.202, 535, 0.43.
- Column names are msms_similarity, n_clus_gt1, per_sing.
- No invented values: all numbers traceable to the article's Table 1 CSF block.

## Limitations

- Values are specific to the CSF dataset reported in the paper; the same metrics must be recomputed for new datasets rather than copied.
- RAMClustR is designed to group features from any chromatographic-spectrometric platform or feature-finding software, but this skill only reproduces published comparison figures, not new benchmark runs.
- No changelog found in the article, so version-specific behavior differences between RAMClustR releases are not documented here.

## Evidence

- [results] CSF rows of Table 1 report MSMS similarity of 0.202 (groupFWHM), 0.177 (groupCorr), 0.043 (groupDen), and 0.382 (ramclustR); n Clus (>1) of 535, 784, 39, and 3248; per Sing of 0.43, 29.56, 0.00, and 15.47.: [evidence span withheld — anchor status: not_found]
- [abstract] RAMClustR is platform-agnostic and works with any feature-finding software, which is why its metrics are compared across grouping methods.: "is sufficiently versatile to group features from any chromatographic-spectrometric platform or feature-finding software"
- [readme] The README provides the R environment in which the benchmark table is assembled and written.: "install_github("cbroeckl/RAMClustR", build_vignettes = TRUE, dependencies = TRUE)"
