---
name: between-group-fold-change-analysis
description: Use when a preprocessed LC-MS feature table (features x samples) has
  labeled biological groups (e.g. 14 lamprey tissues, 94 samples) and you need to
  rank which features are massively enriched in one group versus the pooled rest,
  with FDR-controlled significance.
when_to_use_negative:
- Input is raw instrument data (e.g. mzML/Raw) rather than a preprocessed feature
  table — data preprocessing must happen first via MetaboReport's preprocessing module
- The comparison of interest is between two specific groups of comparable size/structure
  rather than one group versus a pooled remainder, where the >1000-fold 'group-vs-all-others'
  framing does not apply
- Features have already been filtered to a short candidate list — this skill is for
  genome/universe-wide differential abundance screens across all mass features
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetaboReport
  role: Runs the between-group abundance statistical test, FDR adjustment, fold-change
    filtering, and HTML report generation for the metabolomics feature table
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment hosting MetaboReport as an R package and executing the
    analysis helpers in R/fct_*.R
  repo: https://github.com/YonghuiDong/MetaboReport
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:31658262af48ad8ca1eddc545dfe18c600028d47269401a8afebedb103ad0583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Between-group fold-change analysis

## Summary

Identify mass features dramatically enriched in one biological group versus all others by combining a between-group abundance test with a large fold-change cutoff (e.g. >1000x higher) and Benjamini-Hochberg FDR correction, as demonstrated in MetaboReport on lamprey LC-MS spatial metabolomics data.

## When to use

Use when a preprocessed LC-MS feature table (features x samples) has labeled biological groups (e.g. 14 lamprey tissues, 94 samples) and you need to rank which features are massively enriched in one group versus the pooled rest, with FDR-controlled significance.

## When NOT to use

- Input is raw instrument data (e.g. mzML/Raw) rather than a preprocessed feature table — data preprocessing must happen first via MetaboReport's preprocessing module
- The comparison of interest is between two specific groups of comparable size/structure rather than one group versus a pooled remainder, where the >1000-fold 'group-vs-all-others' framing does not apply
- Features have already been filtered to a short candidate list — this skill is for genome/universe-wide differential abundance screens across all mass features

## Inputs

- Preprocessed LC-MS feature abundance table (mass features x samples), negative-ion-mode conditions (e.g. MetaboReport Dataset 1 / cancerCell example data)
- Sample-to-group metadata mapping each of the 94 samples to one of 14 lamprey tissue groups

## Outputs

- Filtered differential abundance table (e.g. differential_abundance_buccal_gland.csv) containing mass features with >1000-fold enrichment and FDR-adjusted P<0.001 (182 rows in the reference case)
- Per-feature between-group test statistics with Benjamini-Hochberg FDR-adjusted P values and fold-change estimates

## How to apply

Load a preprocessed feature table — e.g. MetaboReport's Dataset 1 (cancerCell example data, negative-ion-mode conditions) — into R with sample-to-group annotations. Use MetaboReport's statistical analysis helpers (R/fct_*.R analysis functions) to run a between-group abundance test comparing the group of interest (buccal gland) against the pooled other 13 tissue groups. Apply Benjamini-Hochberg FDR adjustment to the test P values. Compute the per-feature fold change of the target group versus the others and filter to features more than 1000-times higher in the target group, then retain only those passing FDR-adjusted P<0.001. Export the filtered differential abundance table (e.g. differential_abundance_buccal_gland.csv) and validate that the row count matches the reported metric (182 features).

## Related tools

- **MetaboReport** (Runs the between-group abundance statistical test, FDR adjustment, fold-change filtering, and HTML report generation for the metabolomics feature table) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment hosting MetaboReport as an R package and executing the analysis helpers in R/fct_*.R) — https://github.com/YonghuiDong/MetaboReport

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()
```

## Evaluation signals

- The exported filtered table contains exactly 182 rows at FDR-adjusted P<0.001 for the buccal-gland-tissue / negative-ion-mode reference case
- Every retained feature has a computed buccal-gland-vs-other-tissues fold change greater than 1000
- P values have been Benjamini-Hochberg adjusted (FDR), not raw, in the exported table
- The between-group test was run against the pooled other 13 tissue groups, not pairwise tissue comparisons
- MetaboReport's self-contained HTML report includes the statistical results and interactive plots/tables for the analysis

## Limitations

- MetaboReport's supported public package API is intentionally small (run_app()); analysis helpers are internal and their implementation can evolve, so scripted reuse may break between versions
- The reference result (182 features, FDR<0.001) is tied to the specific dataset (94 samples, 14 lamprey tissues, negative ion mode); other datasets will yield different counts
- Results depend on upstream preprocessing quality of the LC-MS feature table; errors in group assignment of samples propagate to the fold-change and FDR results

## Evidence

- [readme] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] `R/fct_*.R` contains pure analysis, input/output, plotting, and report functions.: "`R/fct_*.R` contains pure analysis, input/output, plotting, and report functions."
- [readme] After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app.: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [abstract] we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [methods] - `cancerCell (MetaboReport)`: "- `cancerCell (MetaboReport)`"
