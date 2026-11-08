---
name: spearman-rank-correlation-statistics
description: Use when testing whether detected LC/HRMS peaks from an aligned peak
  table (IDSL.IPA, MS-DIAL, etc.) are significantly associated with a continuous or
  ordinal phenotype (e.g. birth weight) in a cohort.
when_to_use_negative:
- Input is a single replicate or blank-normalized feature table with no phenotype
  metadata to correlate against
- The association model requires covariate adjustment or nonlinear interactions —
  Spearman rank correlation handles only monotone univariate associations
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: IDSL.IPA
  role: Generates the aligned/gap-filled peak-height matrix that is the input to the
    Spearman correlation test
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Implementation environment for the Spearman correlation test and multiple-testing
    adjustment
- name: MS-DIAL
  role: Alternative peak-picking pipeline whose aligned table is processed identically
    for sensitivity comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:490b8aa08cd9687b736321675fb998759762780ed602e69576f0f81a0069aabe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spearman-rank-correlation-statistics

## Summary

Run Spearman rank correlations of IDSL.IPA aligned peak heights against sample phenotypes (e.g. birth weight in the 499-sample MTBLS1684 cord-blood study), adjust p-values for multiple testing (BH/FDR), and count significant peaks (adjusted p ≤ 0.05) to compare data-processing pipelines.

## When to use

Use when testing whether detected LC/HRMS peaks from an aligned peak table (IDSL.IPA, MS-DIAL, etc.) are significantly associated with a continuous or ordinal phenotype (e.g. birth weight) in a cohort.

## When NOT to use

- Input is a single replicate or blank-normalized feature table with no phenotype metadata to correlate against
- The association model requires covariate adjustment or nonlinear interactions — Spearman rank correlation handles only monotone univariate associations

## Inputs

- Aligned peak-height data matrix (IDSL.IPA gap-filled, e.g. MTBLS1684 n=499 samples)
- Sample metadata with phenotype values (e.g. birth weight, kg)
- MS-DIAL aligned peak table for comparison

## Outputs

- Ranked correlation table (peak ID, Spearman rho, raw p, adjusted p, significance flag) written to CSV (e.g. birth_weight_spearman_results.csv)
- Per-pipeline counts of significant peaks (IDSL.IPA-only / common / MS-DIAL-only)

## How to apply

1) Join the IDSL.IPA aligned/gap-filled peak-height matrix to sample metadata on sample IDs. 2) For each peak, compute a Spearman rank correlation of peak height versus the phenotype (e.g. birth weight) in R and adjust p-values for multiple testing (BH/FDR). 3) Filter to adjusted p < 0.05 and rank by rho/p-value. 4) Repeat the identical procedure on a competing pipeline's matrix (e.g. MS-DIAL) and count significant peaks in each; the IDSL.IPA matrix yielded 2.9× more significant peaks (213 IDSL.IPA-only, 401 common, 37 MS-DIAL-only). 5) Export a ranked table with peak ID, rho, raw p, adjusted p, significance flag.

## Related tools

- **IDSL.IPA** (Generates the aligned/gap-filled peak-height matrix that is the input to the Spearman correlation test) — https://github.com/idslme/IDSL.IPA
- **R** (Implementation environment for the Spearman correlation test and multiple-testing adjustment)
- **MS-DIAL** (Alternative peak-picking pipeline whose aligned table is processed identically for sensitivity comparison)

## Examples

```
cor.test(x = peak_heights[, "Peak_A"], y = metadata$birth_weight, method = "spearman")
```

## Evaluation signals

- All reported peaks have adjusted p < 0.05 (BH/FDR)
- Count of significant peaks from IDSL.IPA matrix is ~2.9× the count from MS-DIAL (213 IDSL.IPA-only vs 37 MS-DIAL-only, 401 common)
- Output CSV contains columns: peak ID, rho, raw p, adjusted p, significance flag
- Sample count equals the cohort size (n=499 for MTBLS1684)

## Limitations

- IDSL.IPA aligned matrices require further processing for adducts/fragments identification and statistical filtering before correlation — merged/redundant peaks inflate counts
- Spearman correlation does not establish causality or adjust for confounders (e.g. gestational age, maternal covariates)
- Reported 2.9× enrichment is specific to the MTBLS1684 birth-weight phenotype and may not generalize to other cohorts or phenotypes

## Evidence
