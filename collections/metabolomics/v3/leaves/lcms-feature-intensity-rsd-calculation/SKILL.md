---
name: lcms-feature-intensity-rsd-calculation
description: Use when an aligned LC-MS feature table with replicate sample-intensity
  columns and a 'level' column (PP/MR) is available, and quantitative precision of
  rescued MR features must be benchmarked against conventional peak-picked PP features.
when_to_use_negative:
- The feature table is unaligned (features not yet merged across samples by peak.alignment),
  so replicate intensity columns are not comparable per feature.
- The feature table lacks a 'level' column, so PP and MR features cannot be stratified
  for the paper's precision comparison.
- The dataset has no replicate injections — RSD% cannot be computed from a single
  measurement per feature.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: Produces the aligned feature table (peak.alignment) whose 'level' column (PP/MR)
    is used to stratify features for RSD% computation
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Environment in which RSD% statistics are computed from the aligned feature
    table (JPA requires R version 4.0.0 or above)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:88e77540e91bdbc5f654470a21c8516fe47d4918be5ecf721987122b785ea516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lcms-feature-intensity-rsd-calculation

## Summary

Computes replicate-level relative standard deviation (RSD% = SD/mean intensity × 100) of metabolic features from a JPA-aligned feature table, stratified by feature level (PP = peak picking, MR = MS2 recognition), to assess whether JPA-rescued features are quantitatively precise enough for downstream statistical analysis.

## When to use

Use when an aligned LC-MS feature table with replicate sample-intensity columns and a 'level' column (PP/MR) is available, and quantitative precision of rescued MR features must be benchmarked against conventional peak-picked PP features.

## When NOT to use

- The feature table is unaligned (features not yet merged across samples by peak.alignment), so replicate intensity columns are not comparable per feature.
- The feature table lacks a 'level' column, so PP and MR features cannot be stratified for the paper's precision comparison.
- The dataset has no replicate injections — RSD% cannot be computed from a single measurement per feature.

## Inputs

- Aligned feature table produced by the JPA peak.alignment step, with replicate sample-intensity columns and a 'level' column labeling features as PP or MR
- Sample-group metadata identifying which intensity columns are technical/biological replicates

## Outputs

- rsd_summary.csv containing per-level (PP, MR, overall JPA) average RSD% values and the PP-vs-MR difference
- Pass/fail validation against the <15% precision threshold and the ~3 percentage-point MR-vs-PP expectation

## How to apply

Load the aligned feature table (output of the JPA peak.alignment step) for the dataset of interest — here the original-concentration urine RP(+) run — in R (≥ 4.0.0, JPA installed from GitHub via devtools). Use the 'level' column to stratify features into PP and MR groups. For each feature, compute RSD% = (standard deviation / mean intensity) × 100 across the replicate sample-intensity columns of the aligned table. Group features by level and average RSD% over PP-level features, MR-level features, and all JPA features (PP + MR combined); also compute the MR-minus-PP difference. Write the per-level averages plus the PP-vs-MR difference to rsd_summary.csv, then validate: each group's average RSD% should fall below 15% (the precision threshold reported in the paper), and the JPA-MR average should be approximately 3 percentage points higher than the JPA-PP average.

## Related tools

- **JPA** (Produces the aligned feature table (peak.alignment) whose 'level' column (PP/MR) is used to stratify features for RSD% computation) — https://github.com/HuanLab/JPA.git
- **R** (Environment in which RSD% statistics are computed from the aligned feature table (JPA requires R version 4.0.0 or above))

## Examples

```
Rscript -e 'ft <- read.csv("aligned_table.csv"); rsd <- function(x) sd(x)/mean(x)*100; avg <- tapply(apply(ft[,grep("intensity",names(ft),value=TRUE)],1,rsd), ft$level, mean); write.csv(data.frame(level=names(avg),avg_rsd=avg), "rsd_summary.csv")'
```

## Evaluation signals

- Average RSD% of PP-level features, MR-level features, and all JPA features each fall below 15%, the precision threshold reported in the paper.
- JPA-MR average RSD% is approximately 3 percentage points higher than JPA-PP average RSD%, matching the reported comparison.
- rsd_summary.csv exists and contains exactly the three per-level averages plus the PP-vs-MR difference.
- RSD% is computed only across replicate sample-intensity columns of the aligned table, not across pooled or QC columns unrelated to the replicates.
- Every feature in the table is assigned to exactly one of the PP or MR strata based on the 'level' column, with no unclassified rows.

## Limitations

- The MR average RSD% is expected to be worse (~3 points higher) than PP because MS2-recognition features have bad peak shapes and lower signal intensities by construction.
- Precision is dataset- and mode-specific: the <15% threshold and the ~3-point gap were established on the original-concentration urine RP(+) dataset; other modes or matrices may differ.
- MR feature extraction is only valid for DDA-type data — the README warns not to use the MS2-recognition function when processing full-scan or DIA data sets.

## Evidence

- [abstract] precision of the rescued metabolic features were evaluated: "More importantly, the chromatographic peak shapes, analytical accuracy, and precision of the rescued metabolic features were all evaluated."
- [methods] sample alignment performed after extraction: "Sample alignment is performed after all level features have been extracted."
- [readme] level column labels features: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object.
The "level" column shows the level of each feature."
- [readme] peak.alignment function: "featureTable <- peak.alignment(data = MSdata, bw = 5, minfrac = 0.5, mzwid = 0.015,"
- [readme] PP features are peak-picked: [evidence span withheld — anchor status: not_found]
