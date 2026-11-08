---
name: metabolomics-feature-statistical-testing
description: Use when you have an aligned, quantified feature table (uncorrected or
  batch-corrected apLCMS/XCMS/two-stage output) and two labeled sample groups (e.g.
  oak vs wine yeast cell types) and need to count significant features at a stated
  FDR cutoff such as 0.2.
when_to_use_negative:
- Input is still raw mass-spectrometry files (NetCDF/mzML) rather than an aligned,
  quantified feature table — peak detection and alignment must happen first via apLCMS
  or XCMS
- The comparison involves more than two groups or a paired/repeated-measures design,
  where a two-sample t-test is the wrong model
- The goal is batch effect correction itself rather than downstream differential-abundance
  testing of already-corrected feature tables
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: R
  role: Performs the per-feature two-sample t-tests, Benjamini-Hochberg FDR correction,
    missingness filtering, and counts-table export
- name: apLCMS
  role: Produces the uncorrected LC/MS feature tables (one per parameter setting)
    on which the t-tests are run
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:5c45d89b9b3a391822bf09451b0e6fe4df15f7a13dce73287d4d3c0967b5f8d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-feature-statistical-testing

## Summary

Per-feature two-sample t-testing with Benjamini-Hochberg FDR correction over LC/MS metabolomics feature tables (e.g. ST000868 oak vs wine yeast samples) to count differentially abundant features at FDR <= 0.2, as used to reproduce Fig. 3d of the batch-effect preprocessing paper.

## When to use

Use when you have an aligned, quantified feature table (uncorrected or batch-corrected apLCMS/XCMS/two-stage output) and two labeled sample groups (e.g. oak vs wine yeast cell types) and need to count significant features at a stated FDR cutoff such as 0.2.

## When NOT to use

- Input is still raw mass-spectrometry files (NetCDF/mzML) rather than an aligned, quantified feature table — peak detection and alignment must happen first via apLCMS or XCMS
- The comparison involves more than two groups or a paired/repeated-measures design, where a two-sample t-test is the wrong model
- The goal is batch effect correction itself rather than downstream differential-abundance testing of already-corrected feature tables

## Inputs

- Uncorrected ST000868 feature data matrices from apLCMS (multiple parameter settings), XCMS, and the two-stage preprocessing method
- Cell-type labels assigning each sample to oak or wine yeast strain group

## Outputs

- Per-feature two-sample t-test p-values with Benjamini-Hochberg adjusted FDR values
- Counts of significant features (FDR <= 0.2) per preprocessing method and parameter setting, written to fig3d_significant_feature_counts.csv

## How to apply

Load each uncorrected feature data matrix (one per apLCMS parameter setting, or from XCMS/two-stage preprocessing) for the ST000868 dataset. First filter features to retain only those with <= 33% zero (missing) values within at least one of the two cell types, so the t-test is not dominated by all-missing features. For every retained feature, run a two-sample t-test comparing oak versus wine yeast log-scale intensity values using R. Apply Benjamini-Hochberg FDR correction across features within each parameter setting, then count features with adjusted p-value <= 0.2 per setting and write the counts to fig3d_significant_feature_counts.csv.

## Related tools

- **R** (Performs the per-feature two-sample t-tests, Benjamini-Hochberg FDR correction, missingness filtering, and counts-table export)
- **apLCMS** (Produces the uncorrected LC/MS feature tables (one per parameter setting) on which the t-tests are run) — https://github.com/tianwei-yu/apLCMS

## Examples

```
padj <- p.adjust(apply(ftr,1,function(z) t.test(z[oak],z[wine])$p.value),method='BH'); counts[[setting]] <- sum(padj<=0.2)
```

## Evaluation signals

- Every feature entering the t-test has <= 33% zero values in at least one of the two cell types
- Adjusted p-values lie in [0,1] and are monotone (>= raw p-values) under Benjamini-Hochberg within each parameter setting
- The per-setting counts of FDR <= 0.2 features reproduce the qualitative pattern of Fig. 3d, with the two-stage method detecting more significant features than apLCMS/XCMS
- The output CSV contains one count per parameter setting and method, matching the feature-table inputs supplied

## Limitations

- Without batch effect correction, all methods yielded relatively few significant metabolites at FDR <= 0.2, so counts may be small and qualitative (not quantitative) validation is expected
- Findings depend on uncorrected feature tables; post hoc intensity-only correction cannot fix peak misalignment or incomplete weak signal recovery, so results reflect preprocessing quality as well as statistical testing

## Evidence

- [other] Filter the feature table in R to retain only features with <= 33% zero (missing) values within at least one of the two cell types.: [evidence span withheld — anchor status: not_found]
- [other] t-test and FDR workflow steps: [evidence span withheld — anchor status: not_found]
- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [discussion] two-stage approach context: "We presented a two-stage approach for LC/MS metabolomics data generated in multiple batches."
