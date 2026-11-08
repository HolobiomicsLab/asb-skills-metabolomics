---
name: lc-ms-feature-table-processing
description: Use when you have an LC-MS feature table (samples × metabolite features
  in negative ion mode) plus tissue/group labels, and need to test whether a specific
  tissue (e.g., buccal gland) separates from others in PCA.
when_to_use_negative:
- Input is raw vendor or mzML LC-MS data still needing peak detection/feature finding
  rather than an already-built feature table
- The goal is only generating a packaged HTML report for an already-completed analysis,
  not re-running the tissue-separation PCA itself
- The dataset has no replicate grouping/tissue labels, so the PC1-vs-PC2 tissue-separation
  validation cannot be assessed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: Shiny app / R package performing metabolomics data preprocessing, statistical
    analysis (PCA via internal stats::prcomp-based functions), visualization, and
    HTML reporting
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment; MetaboReport installs as an R package via remotes::install_github
    and its analysis helpers run in R
  repo: https://github.com/YonghuiDong/MetaboReport
- name: ggplot2
  role: Imported plotting dependency used to draw the PC1-vs-PC2 score plot colored
    by tissue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:c2278bc69756d85949948d5a75238f84017194bccfd5f034401c4f4392ae41f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - ggplot2
  - ropls
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lc-ms-feature-table-processing

## Summary

Preprocess and explore an LC-MS spatial metabolomics feature table (e.g., 94 lamprey samples across 14 tissues, negative ion mode) by variance-stabilizing, autoscaling, and running PCA to reveal tissue-level metabolic separation, all within the MetaboReport Shiny app / R package.

## When to use

Use when you have an LC-MS feature table (samples × metabolite features in negative ion mode) plus tissue/group labels, and need to test whether a specific tissue (e.g., buccal gland) separates from others in PCA.

## When NOT to use

- Input is raw vendor or mzML LC-MS data still needing peak detection/feature finding rather than an already-built feature table
- The goal is only generating a packaged HTML report for an already-completed analysis, not re-running the tissue-separation PCA itself
- The dataset has no replicate grouping/tissue labels, so the PC1-vs-PC2 tissue-separation validation cannot be assessed

## Inputs

- LC-MS feature table (samples × metabolite feature intensities) from negative ion mode acquisition, e.g. Supplementary Information File S4 (94 samples, 14 tissues)
- Sample-to-tissue metadata labels (14 tissue categories, including buccal gland)

## Outputs

- Variance-stabilized and autoscaled (mean-center, unit-variance) feature matrix
- Per-sample PC1/PC2 coordinates saved as a PC scores table CSV
- PC1-vs-PC2 PCA score plot (PNG) with points colored by tissue
- Self-contained HTML report from MetaboReport with interactive plots, tables, and statistical results

## How to apply

Load the LC-MS feature table (e.g., from Supplementary Information File S4 with 94 samples across 14 tissues) together with each sample's tissue label. Apply a variance-stabilizing transformation to feature intensities, then autoscale (mean-center, unit-variance) each feature so high-abundance ions do not dominate the PCA. Run principal component analysis on the scaled feature matrix using stats::prcomp via MetaboReport's internal analysis functions, extract per-sample PC1/PC2 coordinates into a PC scores CSV, and draw a PC1-vs-PC2 score plot with ggplot2 colored by tissue. The rationale is that variance-stabilization plus autoscaling is what makes tissue-level metabolic differences visible in the first two PCs, and MetaboReport wraps this preprocessing–statistics–visualization–reporting chain in one Shiny app started with run_app().

## Related tools

- **MetaboReport** (Shiny app / R package performing metabolomics data preprocessing, statistical analysis (PCA via internal stats::prcomp-based functions), visualization, and HTML reporting) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment; MetaboReport installs as an R package via remotes::install_github and its analysis helpers run in R) — https://github.com/YonghuiDong/MetaboReport
- **ggplot2** (Imported plotting dependency used to draw the PC1-vs-PC2 score plot colored by tissue)

## Examples

```
library(MetaboReport); run_app()
```

## Evaluation signals

- Buccal gland samples form a cluster distinctly separated from the other 13 tissues in the PC1–PC2 score plot, matching the separation reported in Section 3 of the article
- PC scores CSV contains exactly one PC1 and PC2 coordinate pair per sample (94 rows for the S4 dataset)
- Each feature in the scaled matrix has mean 0 and variance 1 (mean-center, unit-variance autoscaling)
- The plot is saved as a PNG and the final MetaboReport HTML report includes interactive plots, tables, and statistical results
- The app instance starts cleanly via MetaboReport::run_app() and completes the preprocessing–PCA–reporting chain without error

## Limitations

- The public package API is intentionally small (only run_app()); analysis helpers are internal, so the exact variance-stabilization and autoscaling implementation cannot be called directly and may evolve between versions
- The standalone App download path (SourceForge) targets Windows only; the R package install (remotes::install_github) is the cross-platform alternative
- Findings tied to a specific feature table (S4, 94 samples, 14 tissues, negative ion mode) may not generalize to datasets with different tissues, sample counts, or ionization modes

## Evidence

- [abstract] we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [methods] you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app: "you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app"
- [readme] The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve: "The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve"
- [intro] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
