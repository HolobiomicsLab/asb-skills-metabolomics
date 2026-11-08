---
name: fdr-multiple-testing-correction
description: Use when a between-group abundance test on LC-MS feature tables (e.g.
  94 samples, 14 lamprey tissues, negative ion mode) yields raw per-feature P values
  across thousands of mass features, and a significance cutoff must control the false-discovery
  rate before filtering.
when_to_use_negative:
- Only a handful of hypothesis tests are performed — the correction would be unnecessary
  and overly conservative
- The downstream claim requires per-feature family-wise (e.g. Bonferroni) control
  rather than false-discovery-rate control
- The input is raw spectrometer output (mzML/NetCDF) rather than a quantified feature
  table with per-feature test statistics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3395
tools:
- name: MetaboReport
  role: R Shiny app providing the statistical analysis functions and analysis helpers
    used to run the between-group abundance test whose P values are FDR-adjusted
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment for loading the feature table, applying Benjamini-Hochberg
    adjustment, and exporting the filtered CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:2dc40a77985db0c8d12ff1bd1c7e9941bc511b4363bbbf8b27d3e754fedf7c7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaMapR
  - MetaboReport
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fdr-multiple-testing-correction

## Summary

Benjamini-Hochberg false-discovery-rate adjustment of per-feature P values from metabolomics between-group tests, applied inside MetaboReport's statistical-analysis module before significance cutoffs (e.g. FDR P<0.001) so that thousands of simultaneously tested LC-MS mass features do not yield spurious hits.

## When to use

Use when a between-group abundance test on LC-MS feature tables (e.g. 94 samples, 14 lamprey tissues, negative ion mode) yields raw per-feature P values across thousands of mass features, and a significance cutoff must control the false-discovery rate before filtering.

## When NOT to use

- Only a handful of hypothesis tests are performed — the correction would be unnecessary and overly conservative
- The downstream claim requires per-feature family-wise (e.g. Bonferroni) control rather than false-discovery-rate control
- The input is raw spectrometer output (mzML/NetCDF) rather than a quantified feature table with per-feature test statistics

## Inputs

- Per-feature raw P values from a between-group abundance test on an LC-MS feature table (negative ion mode)
- MetaboReport feature table (e.g. Dataset 1 cancerCell example data)
- Per-feature fold-change values (e.g. buccal gland vs other 13 tissues)

## Outputs

- FDR-adjusted P values per mass feature (Benjamini-Hochberg)
- Filtered differential abundance table (e.g. differential_abundance_buccal_gland.csv) retaining features meeting fold-change and FDR cutoffs

## How to apply

Load the feature table into R (e.g. the cancerCell example dataset shipped with MetaboReport under negative-ion-mode conditions) and run MetaboReport's between-group abundance test via its internal analysis helpers (R/fct_*.R) comparing one tissue group (e.g. buccal gland) against the pooled other 13 tissue groups. Collect the raw per-feature P values for all mass features and apply Benjamini-Hochberg FDR adjustment across the full feature set, not per feature individually, to control the expected fraction of false discoveries. Combine the adjusted P values with the per-feature fold change (e.g. buccal-gland-vs-other-tissues abundance ratio) and filter on both criteria simultaneously — in the lamprey spatial metabolomics case, >1000-fold higher in buccal gland AND FDR-adjusted P<0.001, which retained 182 features. Export the filtered differential abundance table (e.g. differential_abundance_buccal_gland.csv) for downstream reporting.

## Related tools

- **MetaboReport** (R Shiny app providing the statistical analysis functions and analysis helpers used to run the between-group abundance test whose P values are FDR-adjusted) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment for loading the feature table, applying Benjamini-Hochberg adjustment, and exporting the filtered CSV)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()
```

## Evaluation signals

- The filtered differential abundance table contains exactly 182 rows at FDR-adjusted P<0.001, matching the reported buccal-gland metric under negative-ion-mode conditions
- Every retained feature is >1000-fold higher in buccal gland than in the other 13 tissues
- Adjusted P values are monotonic with (>=) the corresponding raw P values and all lie in [0,1]
- Adjustment was applied across the full feature set simultaneously, not per feature or per tissue subset
- Exported CSV includes both the fold-change column and the FDR-adjusted P column for traceability

## Limitations

- MetaboReport's analysis helpers are internal — the supported public package API is intentionally small (run_app()) — so FDR adjustment is performed through the app's statistical analysis module rather than a documented standalone function
- The specific 182-feature/1000-fold/FDR<0.001 result is conditional on the lamprey spatial metabolomics dataset (94 samples, 14 tissues, negative ion mode); other datasets will yield different counts at the same thresholds

## Evidence

- [abstract] MetaboReport abstract workflow span: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] Analysis helpers located in fct files: "`R/fct_*.R` contains pure analysis, input/output, plotting, and report
  functions."
- [readme] Install as R package: "remotes::install_github("YonghuiDong/MetaboReport")"
- [intro] Statistical analysis scope of the app: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
