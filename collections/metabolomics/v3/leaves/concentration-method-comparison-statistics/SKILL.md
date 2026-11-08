---
name: concentration-method-comparison-statistics
description: Use when two or more drift-correction or quantification strategies have
  produced concentration tables for the same targeted LC-MS/MS compounds and you need
  to measure their agreement (slopes ≈ 1 vs ≠ 1, bias, LoA) or attribute differences
  to matrix effects.
when_to_use_negative:
- Input is raw peak-intensity data not yet quantified — this skill compares corrected
  concentrations, and QuantyFey itself does not perform peak integration (integration
  is done beforehand in MultiQuant)
- Only a single drift-correction strategy is available, leaving no method pairs to
  compare
- The goal is drift correction itself (IS correction, QC-based loess, custom/weighted
  bracketing) rather than post-hoc comparison of corrected results
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: QuantyFey
  role: Produces the drift-corrected concentration tables (IS correction, QC-based
    drift correction, custom bracketing, weighted bracketing) that are compared pairwise
    with this skill
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime for fitting Passing-Bablok regression (pairwise-median slope estimator)
    and computing bias, LoA, and Pearson/Spearman correlations
- name: MultiQuant 3.0.3
  role: Upstream peak integration and manual review whose exported peak areas are
    the basis of the quantified concentrations being compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:9ec134525e036c6a8c62fea227c48224c8d3988e855a5bdfe2352b1239c5c884
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# concentration-method-comparison-statistics

## Summary

Statistically compares corrected analyte concentrations produced by alternative drift-correction strategies (IS correction, QC-based loess, custom bracketing, weighted bracketing) using mean bias, limits of agreement, Passing-Bablok regression, and Pearson/Spearman correlation against injection position, to quantify whether strategies agree or reveal matrix effects.

## When to use

Use when two or more drift-correction or quantification strategies have produced concentration tables for the same targeted LC-MS/MS compounds and you need to measure their agreement (slopes ≈ 1 vs ≠ 1, bias, LoA) or attribute differences to matrix effects.

## When NOT to use

- Input is raw peak-intensity data not yet quantified — this skill compares corrected concentrations, and QuantyFey itself does not perform peak integration (integration is done beforehand in MultiQuant)
- Only a single drift-correction strategy is available, leaving no method pairs to compare
- The goal is drift correction itself (IS correction, QC-based loess, custom/weighted bracketing) rather than post-hoc comparison of corrected results

## Inputs

- Corrected concentration tables (one per drift-correction strategy) for the same set of targeted LC-MS/MS compounds, e.g. 25 porcine plasma compounds
- Injection order/position metadata for drift-vs-position correlation checks

## Outputs

- passing_bablok_per_compound.csv (per-compound slopes and intercepts for each strategy pair)
- passing_bablok_summary.csv (aggregated slope/intercept summary per strategy comparison)
- Per-comparison mean bias and limits of agreement (LoA) values
- Pearson and Spearman correlation coefficients of concentration vs injection position

## How to apply

Load the corrected concentration tables (e.g., 25 compounds from four drift-correction strategies) and form all pairwise comparisons of strategies — six pairs for four strategies. For each compound-pair, compute mean bias as the average of relative differences between methods, derive limits of agreement (LoA), and fit Passing-Bablok regression using a pairwise-median slope estimator with intercept at the median in an R implementation (run under R 4.2.0 as used by QuantyFey). Aggregate per-compound slopes and intercepts into per-comparison summary statistics, expecting slopes generally close to 1 except for comparisons involving IS correction, where matrix enhancement made IS-corrected results consistently lower by approximately a factor of two. Additionally correlate concentrations with injection position (Pearson and Spearman) to verify that residual drift effects are absent after correction.

## Related tools

- **QuantyFey** (Produces the drift-corrected concentration tables (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) that are compared pairwise with this skill) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime for fitting Passing-Bablok regression (pairwise-median slope estimator) and computing bias, LoA, and Pearson/Spearman correlations)
- **MultiQuant 3.0.3** (Upstream peak integration and manual review whose exported peak areas are the basis of the quantified concentrations being compared)

## Examples

```
Rscript -e 'library(mcr); fit <- mcreg(conc_qc, conc_is, method.reg="PaBa", method.ci="bootstrap"); print(coef(fit))'
```

## Evaluation signals

- Passing-Bablok slopes are close to 1 for all compounds in comparisons of QC-based, custom bracketing, and weighted bracketing results, deviating from 1 only in comparisons involving IS correction
- IS-corrected concentrations fall consistently lower than other methods by approximately a factor of two, indicating matrix enhancement — a deviation this analysis should surface
- QC-based drift correction and weighted bracketing yield highly similar results, so their pairwise comparison should show slopes near 1 and small LoA despite loess vs linear weighting
- Post-correction Pearson and Spearman correlations of concentration vs injection position should be near zero, confirming residual drift is removed
- Per-compound CSV rows must cover all six pairwise strategy combinations for the 25 compounds before aggregation

## Limitations

- Unsuitable internal-standard selection (e.g., 4-hydroxy-proline and methionine sulfoxide, whose analogue trends diverged from the analytes) introduced biases, so strategy agreement is compound-specific and strategy choice must be made per compound
- Weighted bracketing performed worse than loess-based approaches in the simulation, and custom/weighted bracketing showed variable performance across compounds, so comparable results cannot be assumed for every compound
- This skill presumes concentrations were generated with external calibration by QuantyFey (Windows-only standalone, or R 4.2.x / 4.5.x / Apptainer); it does not perform integration or correction itself

## Evidence

- [abstract] the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [abstract] IS-corrected results lower by factor of two: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods. This suggests matrix enhancement upon LC-MS/MS analysis."
- [abstract] QC vs weighted bracketing similarity: "Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the"
- [readme] QuantyFey purpose: "QuantyFey is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**."
- [readme] no integration capabilities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
