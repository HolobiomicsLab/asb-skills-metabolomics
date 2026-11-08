---
name: negative-ion-mode-lc-ms-data-interpretation
description: Use when you have a negative-ion-mode LC-MS feature table with tissue/group
  labels (e.g. 94 samples across 14 tissues) and need to determine whether a specific
  tissue separates from the others in PCA space.
when_to_use_negative:
- The data is raw instrument output (e.g. vendor RAW or mzML spectra) rather than
  a processed feature/intensity table with sample-by-tissue metadata
- The analysis goal is feature annotation or metabolite identification rather than
  exploratory tissue-separation assessment by PCA
- Only two groups are being compared and a univariate test is sufficient — multivariate
  PCA separation is designed for many-group exploration (e.g. 14 tissues)
edam_operation: http://edamontology.org/operation_2939
tools:
- name: MetaboReport
  role: R Shiny app used to load the feature table, run PCA via internal analysis
    functions, visualize the PC1–PC2 score plot, and generate the HTML report
  repo: https://github.com/YonghuiDong/MetaboReport
- name: stats
  role: Provides stats::prcomp, used internally by MetaboReport to run principal component
    analysis on the scaled feature matrix
- name: ggplot2
  role: Used to draw the PC1-vs-PC2 score plot with points colored by tissue and to
    export it as a PNG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:380e4d69268f595d7f1cc2c00e1699ab0a7234c42b6da7f9e659b1687df3f506
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# negative-ion-mode-lc-ms-data-interpretation

## Summary

Interpret negative-ion-mode LC-MS spatial metabolomics feature tables by applying variance-stabilizing transformation and autoscaling, then running PCA (PC1–PC2) to reveal tissue-level metabolic separation, as demonstrated for the lamprey buccal gland dataset (94 samples, 14 tissues) analyzed with MetaboReport.

## When to use

Use when you have a negative-ion-mode LC-MS feature table with tissue/group labels (e.g. 94 samples across 14 tissues) and need to determine whether a specific tissue separates from the others in PCA space.

## When NOT to use

- The data is raw instrument output (e.g. vendor RAW or mzML spectra) rather than a processed feature/intensity table with sample-by-tissue metadata
- The analysis goal is feature annotation or metabolite identification rather than exploratory tissue-separation assessment by PCA
- Only two groups are being compared and a univariate test is sufficient — multivariate PCA separation is designed for many-group exploration (e.g. 14 tissues)

## Inputs

- Negative-ion-mode LC-MS feature/intensity table (e.g. Supplementary Information File S4; 94 samples, 14 tissues)
- Per-sample tissue (group) label metadata table

## Outputs

- PC scores table (per-sample PC1 and PC2 coordinates) saved as CSV
- PC1-vs-PC2 PCA score plot colored by tissue, saved as PNG
- Optionally, a self-contained HTML report with interactive plots and statistical results via MetaboReport

## How to apply

Load the LC-MS feature table (e.g. Supplementary Information File S4) together with per-sample tissue labels. Apply a variance-stabilizing transformation to the feature intensities, then autoscale each feature (mean-center, unit-variance) so that high-abundance ions do not dominate the analysis. Run principal component analysis on the scaled matrix — in the MetaboReport workflow this uses stats::prcomp via the app's internal analysis functions. Extract per-sample PC1 and PC2 scores into a CSV table and draw the PC1-vs-PC2 score plot colored by tissue (e.g. with ggplot2). The rationale for scaling is that untransformed, unscaled negative-ion-mode intensity data is dominated by high-abundance features, which can mask genuine tissue-specific metabolic signatures.

## Related tools

- **MetaboReport** (R Shiny app used to load the feature table, run PCA via internal analysis functions, visualize the PC1–PC2 score plot, and generate the HTML report) — https://github.com/YonghuiDong/MetaboReport
- **stats** (Provides stats::prcomp, used internally by MetaboReport to run principal component analysis on the scaled feature matrix)
- **ggplot2** (Used to draw the PC1-vs-PC2 score plot with points colored by tissue and to export it as a PNG)

## Examples

```
library(MetaboReport); run_app()
```

## Evaluation signals

- The buccal gland samples form a cluster separated from the other 13 tissues in the PC1–PC2 score plot, matching the separation reported in Section 3 of the article
- Feature intensities are variance-stabilized and each feature is autoscaled (mean-centered, unit-variance) before prcomp — verify no raw intensities enter the model
- PC scores table contains one PC1 and PC2 coordinate per sample for all 94 samples and is exported as CSV
- The score plot colors points by tissue label and all 14 tissues are represented
- MetaboReport's HTML report includes the interactive PCA plot and statistical results

## Limitations

- MetaboReport's public package API is intentionally small (run_app()); analysis helpers are internal, so programmatic reuse of the PCA functions depends on the app's modules rather than a stable exported API
- PCA separation is an exploratory finding; a distinct cluster in the PC1–PC2 score plot does not by itself identify which metabolites drive the separation
- The demonstrated dataset is specific: negative ion mode, 94 samples, 14 lamprey tissues — separations observed there may not transfer to other ion modes or sample designs

## Evidence

- [supplementary] PCA via prcomp on transformed, autoscaled feature table; buccal gland separates from 13 other tissues: "The PCA score plot generated by MetaboReport clearly shows that the buccal gland is distinctly separated from all other tissues, indicating a unique metabolic profile."
- [supplementary] Autoscaling rationale: [evidence span withheld — anchor status: not_found]
- [abstract] MetaboReport scope: "MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] Installation and app startup: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [readme] Internal analysis helpers: "The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve"
