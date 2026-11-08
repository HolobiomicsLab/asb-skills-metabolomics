---
name: lc-ms-concentration-result-correlation-analysis
description: Use when a targeted LC-MS/MS quantification experiment has been processed
  with multiple drift-correction/quantification strategies and you must quantify their
  agreement (e.g. test whether non-IS strategies correlate at Pearson r > 0.9).
when_to_use_negative:
- The data are raw peak areas or intensity drifts that still need correction — QuantyFey
  works as additional software to already integrated mass spectrometry data and does
  not provide integration capabilities; first run quantification, then correlate.
- The goal is fitting or optimizing calibration regression models (linear/quadratic,
  weights, standard levels) rather than comparing agreement between already-computed
  concentration results.
- Only one drift-correction strategy was applied, leaving no pairs of method columns
  to correlate.
edam_operation: http://edamontology.org/operation_3463
tools:
- name: QuantyFey
  role: Produces the per-method concentration tables (IS correction, QC-based drift
    correction, custom bracketing, weighted bracketing) whose columns are correlated
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Used to compute pairwise Pearson correlation coefficients (cor(method='pearson'))
    between the method concentration columns
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:1ebb41f47e98e3fbb8f401e586775f6eff17a2e9d02b16503ad19e27d9b626cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3463
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lc-ms-concentration-result-correlation-analysis

## Summary

Cross-method agreement analysis for targeted LC-MS/MS quantification: compute pairwise Pearson correlations between concentration tables produced by different drift-correction strategies (default bracketing, QC-based drift correction, custom bracketing, weighted bracketing, IS correction) to determine which strategies yield concordant results.

## When to use

Use when a targeted LC-MS/MS quantification experiment has been processed with multiple drift-correction/quantification strategies and you must quantify their agreement (e.g. test whether non-IS strategies correlate at Pearson r > 0.9).

## When NOT to use

- The data are raw peak areas or intensity drifts that still need correction — QuantyFey works as additional software to already integrated mass spectrometry data and does not provide integration capabilities; first run quantification, then correlate.
- The goal is fitting or optimizing calibration regression models (linear/quadratic, weights, standard levels) rather than comparing agreement between already-computed concentration results.
- Only one drift-correction strategy was applied, leaving no pairs of method columns to correlate.

## Inputs

- QuantyFey-exported concentration results table (one column per drift-correction method, one row per compound)
- Concentration values for the compounds analyzed (e.g. 25 porcine plasma analytes) from default bracketing, QC-based drift correction, custom bracketing, and weighted bracketing (and optionally IS correction)

## Outputs

- Pairwise Pearson correlation coefficient matrix (correlation of each method against every other method across compounds)
- Assessment of which drift-correction strategies agree strongly (r > 0.9) and which diverge

## How to apply

Export the concentration results table from QuantyFey (e.g. 25 porcine plasma compounds) with one column per method: default bracketing, QC-based drift correction, custom bracketing, and weighted bracketing. Load the table into R 4.2.0 and compute the pairwise Pearson correlation coefficients across compounds using cor(method='pearson'), assembling the resulting four-by-four (or five-by-five, including IS correction) correlation matrix. Inspect pairwise r values: agreement of the non-IS strategies (QC-based drift correction, custom bracketing, weighted bracketing) at r > 0.9 indicates strong linear concordance, consistent with the paper's finding that QC-based drift correction and weighted bracketing yielded highly similar results despite linear weighting versus non-linear loess. Treat lower correlations for default bracketing and IS correction as evidence those methods disagree with the others — in this study IS-corrected concentrations were consistently lower by approximately a factor of two, suggesting matrix enhancement. Optionally complement Pearson correlation with mean bias, limits of agreement, and Passing-Bablok regression as used in the paper's method evaluation.

## Related tools

- **QuantyFey** (Produces the per-method concentration tables (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) whose columns are correlated) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Used to compute pairwise Pearson correlation coefficients (cor(method='pearson')) between the method concentration columns)

## Examples

```
R: cor(results[, c("default_bracketing","QC_drift","custom_bracketing","weighted_bracketing")], method='pearson')
```

## Evaluation signals

- The three non-IS correction strategies (QC-based drift correction, custom bracketing, weighted bracketing) mutually correlate at Pearson r > 0.9, as reported in the paper.
- Default bracketing and IS-correction columns show lower Pearson correlation with the other quantification methods.
- The correlation matrix is square and symmetric with diagonal values of 1 and contains n-choose-2 pairwise coefficients for n methods.
- IS-corrected concentrations are consistently lower than other methods by roughly a factor of two, so its correlations should be depressed relative to non-IS pairs.

## Limitations

- Pearson correlation measures linear agreement only; the paper notes QC-based drift correction uses non-linear loess while weighted bracketing applies linear weighting, yet the two yielded highly similar results — custom and weighted bracketing showed variable performance across compounds, so per-compound agreement can vary.
- A low correlation does not by itself identify which method is correct; in this study the IS-corrected values were shifted by about a factor of two (suggesting matrix enhancement), and unsuitable IS selection (e.g. 4-hydroxy-proline and methionine sulfoxide) introduced biases.
- QuantyFey is compatible with Windows operating systems only for the standalone version; other platforms require the Apptainer version or R 4.2.x/4.5.x launch.

## Evidence

- [abstract] Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [abstract] Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the: "Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the"
- [methods] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
- [readme] QuantyFey is a Shiny application for the visualization, analysis, and quantification of mass spectrometry (MS) data using external calibration.: "**QuantyFey** is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**."
