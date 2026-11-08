---
name: spearman-injection-order-analysis
description: Use when a quantified concentration table from a targeted LC-MS/MS sequence
  (e.g. porcine plasma) must be checked for residual injection-order drift after applying
  a drift-correction strategy, or when comparing correction strategies for compound-specific
  residual drift.
when_to_use_negative:
- When peak integration or chromatographic data processing is still needed — QuantyFey
  works only as additional software on already integrated mass spectrometry data and
  provides no integration capabilities
- When the goal is overall quantification-accuracy comparison between methods (mean
  bias, limits of agreement, Passing-Bablok regression) rather than detection of residual
  injection-order drift
tools:
- name: R 4.2.0
  role: Computes the Spearman rank correlation between injection position and concentration
    via stats::cor.test with method="spearman" for each compound and strategy
- name: QuantyFey
  role: Generates the quantified concentration tables under each drift-correction
    strategy (IS correction, QC-based drift correction, custom bracketing, weighted
    bracketing) that are then tested for residual drift
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:61edebc50ae43e21adcb52d22e7ef63d915b556eeee8d7e72366ca4152c66efc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QuantyFey
  - R 4.2.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spearman-injection-order-analysis

## Summary

Assesses residual intensity drift in targeted LC-MS/MS quantification by computing Spearman rank correlation between injection position and determined concentration for each compound under each drift-correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing), using R stats::cor.test.

## When to use

Use when a quantified concentration table from a targeted LC-MS/MS sequence (e.g. porcine plasma) must be checked for residual injection-order drift after applying a drift-correction strategy, or when comparing correction strategies for compound-specific residual drift.

## When NOT to use

- When peak integration or chromatographic data processing is still needed — QuantyFey works only as additional software on already integrated mass spectrometry data and provides no integration capabilities
- When the goal is overall quantification-accuracy comparison between methods (mean bias, limits of agreement, Passing-Bablok regression) rather than detection of residual injection-order drift

## Inputs

- Quantified concentration table produced under each drift-correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing)
- Raw, uncorrected concentrations for baseline drift assessment
- Injection order of the LC-MS/MS sequence (sample positions)

## Outputs

- Table of Spearman correlation coefficients and p-values per compound per correction strategy
- Flags marking strategies/compounds with significant residual drift (p <= 0.05)
- Baseline drift assessment of raw uncorrected concentrations

## How to apply

Load the quantified concentration table produced under each correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) together with the injection order of the sequence. For each compound and each strategy, compute Spearman rank correlation between injection position and determined concentration in R with stats::cor.test(method="spearman"). First repeat the test on raw, uncorrected concentrations to establish the baseline residual drift against which corrected results are judged. For compounds with IS-analogue mismatch (4-hydroxy-proline and methionine sulfoxide vs proline and methionine), compute the correlation under IS correction specifically to reveal the bias that increases values at the sequence end. Tabulate correlation coefficients and p-values per compound per strategy and flag strategies with significant residual drift.

## Related tools

- **R 4.2.0** (Computes the Spearman rank correlation between injection position and concentration via stats::cor.test with method="spearman" for each compound and strategy)
- **QuantyFey** (Generates the quantified concentration tables under each drift-correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) that are then tested for residual drift) — https://github.com/CDLMarkus/QuantyFey/releases/

## Examples

```
cor.test(injection_order, concentration, method="spearman")
```

## Evaluation signals

- A strategy is judged drift-free when its Spearman correlation of concentration vs injection position is non-significant (p > 0.05) for a given compound
- The raw, uncorrected concentrations should show significant Spearman correlation with injection position, establishing the baseline drift that corrected strategies must remove
- Under IS correction, 4-hydroxy-proline and methionine sulfoxide should show persistent drift (increased values at the sequence end) due to differing trends vs their analogues proline and methionine
- QC-based and IS-based correction should yield non-significant residual-drift correlations, consistent with the paper's finding that both significantly reduced drift effects
- Per-compound results must be tabulated so compound-dependent performance of custom and weighted bracketing is visible rather than averaged away

## Limitations

- IS correction introduces bias for compounds whose analyte and IS analogue show differing drift trends (e.g. 4-hydroxy-proline and methionine sulfoxide vs proline and methionine), so a non-significant Spearman result under one strategy does not generalize across compounds
- Custom and weighted bracketing showed variable, compound-dependent performance, so absence of significant drift for one compound does not certify the strategy drift-free overall

## Evidence

- [methods] Evaluation of drift correction methods via Spearman correlation of concentration vs injection position: [evidence span withheld — anchor status: not_found]
- [abstract] Drift-reduction finding supporting the p > 0.05 pass criterion: "Both QC-based and IS-based correction significantly reduced drift effects."
- [abstract] IS-analogue mismatch compounds with persistent drift: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases, leading to increased values at the"
- [readme] Bracketing strategies producing the concentration tables under test: "Weighted Bracketing Weighting of calibration curves based on the position of the samples between calibration curves."
