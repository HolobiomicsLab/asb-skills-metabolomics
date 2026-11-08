---
name: qc-threshold-verification-metabolomics
description: Use when an aligned feature table with replicate sample-intensity columns
  is available and you must confirm that rescued (MR) and peak-picked (PP) features
  meet a precision threshold, e.g. RSD% < 15%, before downstream statistics.
when_to_use_negative:
- The dataset has no replicate injections or no aligned feature table (e.g. raw mzXML
  files that still need JPA feature extraction and peak.alignment first)
- The goal is to evaluate LOD or dilution-related feature rescue rather than replicate
  precision
- Features come from a different processing tool that does not label levels as PP/MR,
  so the per-level stratification is undefined
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: JPA
  role: Source of the aligned feature table with PP/MR level labels (peak.alignment
    output) on which RSD% is computed
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Environment for computing per-feature RSD% across replicate intensity columns
    and per-level averages; JPA requires R version 4.0.0 or above
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:70d8b07204ca6016dab458b8343fc6dc312930147deed3c545a4f2a163b4071e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# qc-threshold-verification-metabolomics

## Summary

Verifies the analytical precision of extracted LC-MS metabolic features by computing per-feature RSD% across replicate injections, stratifying by JPA feature level (PP vs MR), and checking group averages against the paper's 15% precision threshold.

## When to use

Use when an aligned feature table with replicate sample-intensity columns is available and you must confirm that rescued (MR) and peak-picked (PP) features meet a precision threshold, e.g. RSD% < 15%, before downstream statistics.

## When NOT to use

- The dataset has no replicate injections or no aligned feature table (e.g. raw mzXML files that still need JPA feature extraction and peak.alignment first)
- The goal is to evaluate LOD or dilution-related feature rescue rather than replicate precision
- Features come from a different processing tool that does not label levels as PP/MR, so the per-level stratification is undefined

## Inputs

- Aligned JPA feature table (output of peak.alignment) with a 'level' column labeling features as PP or MR and replicate sample-intensity columns

## Outputs

- rsd_summary.csv containing per-level average RSD% values (PP, MR, all JPA) and the PP-vs-MR difference
- Per-feature RSD% values across replicates

## How to apply

Load the aligned feature table produced by the JPA peak.alignment step in R (R >= 4.0.0) and use its 'level' column to stratify features into PP (peak picking) and MR (MS2 recognition) groups. For each feature, compute RSD% = (standard deviation / mean intensity) x 100 across the replicate sample-intensity columns. Average the RSD% separately over PP-level features, MR-level features, and all JPA features combined, then write the per-level averages plus the PP-vs-MR difference to rsd_summary.csv. Validate against the paper's criteria: every group average must fall below 15% RSD, and the JPA-MR average should be approximately 3 percentage points higher than the JPA-PP average, reflecting the slightly lower precision of MS2-rescued features.

## Related tools

- **JPA** (Source of the aligned feature table with PP/MR level labels (peak.alignment output) on which RSD% is computed) — https://github.com/HuanLab/JPA.git
- **R** (Environment for computing per-feature RSD% across replicate intensity columns and per-level averages; JPA requires R version 4.0.0 or above)

## Examples

```
rsd <- apply(ft[, grep('Intensity', names(ft))], 1, function(x) sd(x)/mean(x)*100); tapply(rsd, ft$level, mean)
```

## Evaluation signals

- Average RSD% for PP-level, MR-level, and all-JPA feature groups each fall below 15%, the precision threshold reported in the paper
- JPA-MR average RSD% is approximately 3 percentage points higher than the JPA-PP average RSD%
- Every feature's RSD% is computed across the replicate sample-intensity columns of the aligned table (not between levels)
- rsd_summary.csv exists and contains one average RSD% per level plus the PP-vs-MR difference
- Feature counts per level match the 'level' column values in the aligned feature table (only PP and MR labels expected)

## Limitations

- The 15% precision threshold is the specific acceptance criterion reported in this paper; other studies may require different cutoffs
- MR (MS2-recognized) features are expected to be intrinsically less precise (~3 percentage points higher RSD%) than PP features, so uniform thresholds across levels may misclassify rescued features
- Precision assessment requires replicate measurements; it cannot be applied to single-sample feature tables

## Evidence

- [abstract] precision threshold and PP/MR comparison: "More importantly, the chromatographic peak shapes, analytical accuracy, and precision of the rescued metabolic features were all evaluated."
- [readme] aligned table provenance: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [readme] alignment precedes table use: "Note: for multi-sample analysis, sample alignment is performed after feature extraction."
