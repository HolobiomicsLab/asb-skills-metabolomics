---
name: metabolomics-replicate-precision-assessment
description: Use when you have an aligned feature table from JPA's peak.alignment
  step with replicate sample-intensity columns and need to quantify analytical precision
  of peak-picked (PP) versus MS2-recognized (MR) features before statistical testing.
when_to_use_negative:
- The feature table has not been through JPA's peak.alignment step (unaligned per-sample
  tables lack a common feature index and 'level' column)
- The dataset has no replicate samples, so RSD% cannot be computed
- You need to evaluate LOD, peak shape, or feature-rescue rate rather than replicate
  precision — use the paper's dilution-series or LOD workflows instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: R
  role: Computing per-feature RSD% (sd/mean x 100) over replicate intensity columns
    and writing rsd_summary.csv
  repo: https://github.com/HuanLab/JPA.git
- name: JPA
  role: Produces the aligned feature table (peak.alignment) whose 'level' column stratifies
    features into PP and MR groups for the precision comparison
  repo: https://github.com/HuanLab/JPA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:4df937897228e705816f021653a0b91f5d8ce8e62665eaf5e0f13ffe15c1db80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-replicate-precision-assessment

## Summary

Computes replicate relative standard deviation (RSD%) of aligned LC-MS metabolic features, stratified by JPA extraction level (PP vs MR), to verify that rescued features meet the precision threshold (<15% RSD) required for downstream statistical analysis.

## When to use

Use when you have an aligned feature table from JPA's peak.alignment step with replicate sample-intensity columns and need to quantify analytical precision of peak-picked (PP) versus MS2-recognized (MR) features before statistical testing.

## When NOT to use

- The feature table has not been through JPA's peak.alignment step (unaligned per-sample tables lack a common feature index and 'level' column)
- The dataset has no replicate samples, so RSD% cannot be computed
- You need to evaluate LOD, peak shape, or feature-rescue rate rather than replicate precision — use the paper's dilution-series or LOD workflows instead

## Inputs

- Aligned feature table (CSV/dataframe) produced by JPA peak.alignment, with a 'level' column (PP/MR) and replicate sample-intensity columns
- Replicate grouping information identifying QC/biological replicate sample columns in the aligned table

## Outputs

- rsd_summary.csv containing per-level average RSD% values (PP, MR, all JPA) and the PP-vs-MR difference
- Per-feature RSD% values across replicate samples

## How to apply

Load the aligned feature table (the dataframe output of JPA's peak.alignment function) in R and use the 'level' column to stratify features into PP (conventional peak picking) and MR (MS2 recognition) groups. For each feature, compute RSD% = (standard deviation / mean intensity) x 100 across the replicate sample-intensity columns. Average RSD% within each level group (PP, MR, and all JPA features combined) and write the per-level averages plus the PP-vs-MR difference to rsd_summary.csv. Validate against the paper's criteria: every group's average RSD% should fall below 15% (the precision threshold reported in the article), and the JPA-MR average should be approximately 3 percentage points higher than JPA-PP's, reflecting the slightly lower precision of MS2-rescued features.

## Related tools

- **R** (Computing per-feature RSD% (sd/mean x 100) over replicate intensity columns and writing rsd_summary.csv) — https://github.com/HuanLab/JPA.git
- **JPA** (Produces the aligned feature table (peak.alignment) whose 'level' column stratifies features into PP and MR groups for the precision comparison) — https://github.com/HuanLab/JPA

## Examples

```
Rscript -e 'ft <- read.csv("aligned_featureTable.csv"); rsd <- function(x) 100*sd(x, na.rm=TRUE)/mean(x, na.rm=TRUE); agg <- tapply(rowMeans(ft[,samples]), ft$level, function(v) NA); rsd_summary <- aggregate(. ~ level, data=ft, FUN=function(col) mean(apply(col, 1, rsd))); write.csv(rsd_summary, "rsd_summary.csv")'
```

## Evaluation signals

- Average RSD% of PP-level features is below 15% (the paper's precision threshold)
- Average RSD% of MR-level features is below 15% and approximately 3 percentage points higher than the PP average
- Combined JPA (PP + MR) average RSD% is also below 15%
- Every feature's RSD% is computed from at least one replicate intensity column of the aligned table (no features silently dropped)
- rsd_summary.csv contains exactly one row per stratification level (PP, MR, JPA overall) plus the PP-vs-MR difference

## Limitations

- RSD% for MR features is expected to be ~3 points higher than PP features because MS2-recognized features were originally missed by peak picking due to low signal intensity or poor peak shape
- Precision below 15% is the criterion for suitability of downstream statistical analysis; features above this threshold may not be reliable
- The precision evaluation was demonstrated on the original-concentration urine RP(+) dataset; other modes or matrices may behave differently

## Evidence

- [readme] RSD stratification rationale: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object.
The "level" column shows the level of each feature."
- [abstract] precision evaluation scope: "More importantly, the chromatographic peak shapes, analytical accuracy, and precision of the rescued metabolic features were all evaluated."
- [abstract] why MR features have lower precision: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
- [readme] alignment precedes downstream evaluation: "Note: for multi-sample analysis, sample
alignment is performed after feature extraction."
