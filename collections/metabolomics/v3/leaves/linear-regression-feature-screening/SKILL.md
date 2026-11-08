---
name: linear-regression-feature-screening
description: Use when an untargeted LC-MS feature table (m/z rows, samples as columns)
  must be screened for features significantly altered by a binary condition, with
  batch-corrected vs raw comparisons.
when_to_use_negative:
- Data has not yet been corrected for drift across batches — the article's workflow
  applies this screening AFTER dbnorm batch-effect correction (ber, ComBat, lowess),
  not on unprocessed intensities.
- The experiment has more than two conditions or a continuous covariate design where
  a simple two-group linear regression does not capture the biological contrast of
  interest.
- The goal is to select the batch-correction model itself rather than find differential
  features — use dbnormSCORE / adj-R2 scoring instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: dbnorm
  role: Provides the batch-corrected data matrix (via ber or other models) that serves
    as the input to the per-feature linear regression screening, and supports the
    exploratory PCA used to confirm batch structure before testing
  repo: github.com/NBDZ/dbnorm
- name: R
  role: Environment in which the per-feature linear regression models, BH adjustment,
    and volcano plotting are executed
- name: ggplot2
  role: R package used to render the volcano plot of Log2FC versus -log10(adjusted
    p-value) highlighting the significant feature subset
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:6e6692a560f55b0e218b5b3aafcfc2275505d80abf570be1a7786bbfac91052d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dbnorm
  - ggplot2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# linear-regression-feature-screening

## Summary

Per-feature linear regression of metabolomics intensities against a biological condition (e.g., HFD vs control) to estimate Log2 fold-change and p-values, followed by BH adjustment and screening with Log2FC > 1.5 and adjusted p < 0.05 to define a volcano-plot-highlighted differential feature list.

## When to use

Use when an untargeted LC-MS feature table (m/z rows, samples as columns) must be screened for features significantly altered by a binary condition, with batch-corrected vs raw comparisons.

## When NOT to use

- Data has not yet been corrected for drift across batches — the article's workflow applies this screening AFTER dbnorm batch-effect correction (ber, ComBat, lowess), not on unprocessed intensities.
- The experiment has more than two conditions or a continuous covariate design where a simple two-group linear regression does not capture the biological contrast of interest.
- The goal is to select the batch-correction model itself rather than find differential features — use dbnormSCORE / adj-R2 scoring instead.

## Inputs

- Batch-corrected (or raw) LC-MS metabolomics feature-intensity matrix in .csv format (samples in rows, features in columns, batch level in first column)
- Sample metadata / condition labels (e.g., HFD vs control, 8-week diet groups)

## Outputs

- Table of per-feature Log2 fold-changes, raw and BH-adjusted p-values
- Filtered list of significant metabolites (Log2FC > 1.5, adj-p < 0.05) exported as CSV
- Volcano plot (Log2FC vs -log10(adj-p)) as PNG highlighting significant features

## How to apply

Take the feature-intensity matrix (samples in rows, features in columns, batch level in the first column, log2-scaled) and fit a linear regression model per metabolic feature to estimate the fold change and p-value of the diet (or treatment) effect. Adjust p-values with Benjamini-Hochberg correction for multiple testing, then select features satisfying Log2FC > 1.5 and adjusted p-value < 0.05 as significant. Plot Log2FC against -log10(adjusted p) as a volcano plot (ggplot2) highlighting the significant subset, and run the identical regression on raw data to compare whether ber-corrected data surfaces additional significant metabolites. In the HFD study, this comparison showed that most significant features were lipids and that several metabolites were detected as HFD-altered only after batch correction with the best-scoring dbnorm model.

## Related tools

- **dbnorm** (Provides the batch-corrected data matrix (via ber or other models) that serves as the input to the per-feature linear regression screening, and supports the exploratory PCA used to confirm batch structure before testing) — github.com/NBDZ/dbnorm
- **R** (Environment in which the per-feature linear regression models, BH adjustment, and volcano plotting are executed)
- **ggplot2** (R package used to render the volcano plot of Log2FC versus -log10(adjusted p-value) highlighting the significant feature subset)

## Examples

```
R: fit lm per feature on the corrected matrix; e.g. `res <- apply(f, 1, function(x) summary(lm(x ~ diet))$coefficients[2,])`, then BH-adjust with `p.adjust(res[,4], method="BH")` and plot with ggplot2.
```

## Evaluation signals

- Every reported significant feature must satisfy both criteria simultaneously: Log2FC > 1.5 and BH-adjusted p-value < 0.05.
- The adjusted p-value column must reflect Benjamini-Hochberg correction over all tested metabolic features, not raw per-feature p-values.
- Volcano plot axes must correspond to Log2FC (x) and -log10(adjusted p-value) (y), with the significant subset visually distinguished from the null features.
- A parallel run on raw data should be produced for comparison; features significant only after ber-correction are expected to appear predominantly as lipids in the HFD dataset.
- Output CSV row count must match the number of features passing both thresholds, and each row should carry its Log2FC and adjusted p-value.

## Limitations

- The Log2FC > 1.5 / adj-p < 0.05 cutoffs were calibrated for the mouse sc-AT HFD untargeted LC-MS dataset; other study designs may require recalibration.
- Batch correction choice materially changes the significant-feature list — lowess and nonparametric ComBat leave residual batch variation (up to 78% and 60% respectively), so screening on data corrected with a poorly-fitting model can be misleading.
- Missing values must be imputed (e.g., with dbnorm's emvd/emvf, which replace Zero/NA with the lowest detected value) before regression, otherwise feature-wise models may fail or be biased.
- The significance list should be interpreted with downstream pathway context (e.g., MSEA annotation of HFD-influenced metabolites) rather than as standalone biological conclusions.

## Evidence

- [results] Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level.: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [readme] Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked.: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features"
- [readme] This function allows you to estimate missing values (Zero or/and NA values) by the lowest detected value in the entire experiment.: "This function allows you to estimate missing values (Zero or/and NA values) by the lowest detected value"
- [results] Applying the significant criteria of Log2FC > 1.5 and adjusted p-value < 0.05, ber-corrected data yielded a more exhaustive candidate list for HFD effects than raw data, with several metabolites (mostly lipids) significantly affected by HFD only after batch correction.: [evidence span withheld — anchor status: not_found]
- [results] we next annotated the metabolites that were significantly influenced by HFD to functional pathways by using metabolite set enrichment analysis (MSEA): "annotated the metabolites that were significantly influenced by HFD to functional pathways by using metabolite set enrichment analysis (MSEA)"
