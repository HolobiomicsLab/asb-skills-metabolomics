---
name: lc-ms-drift-residual-correlation-testing
description: Use when a targeted LC-MS/MS sequence has been quantified under one or
  more drift correction strategies (IS correction, QC-based drift correction, custom
  bracketing, weighted bracketing) and you need to test whether injection-order-dependent
  bias remains.
when_to_use_negative:
- The data are integrated peak areas rather than final quantified concentrations —
  QuantyFey performs quantification first and does not provide peak integration capabilities
  itself
- The goal is to compare correction methods on accuracy metrics (mean bias, limits
  of agreement, Passing-Bablok regression) rather than to detect residual injection-order
  drift
- The input comes from untargeted/untreated MS data where no external-calibration
  quantification table and no defined injection order exist
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QuantyFey
  role: Produces the quantified concentration tables under each drift correction strategy
    (IS correction, QC-based drift correction, custom bracketing, weighted bracketing)
    that feed the Spearman drift test
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime used to run stats::cor.test with method="spearman" on injection position
    vs determined concentration; QuantyFey itself is implemented in R (v4.2.0)
- name: MultiQuant 3.0.3
  role: Upstream vendor software in which peak integration and manual review were
    performed before peak areas and RTs were exported for quantification
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:480551b9094b6c86873013228ed1561293b91137ed4c7cca4c5c5d61676e7a48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-ms-drift-residual-correlation-testing

## Summary

Quantifies residual intensity drift after LC-MS/MS drift correction by computing Spearman rank correlation between injection position and determined concentration per compound, per correction strategy, using stats::cor.test in R.

## When to use

Use when a targeted LC-MS/MS sequence has been quantified under one or more drift correction strategies (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) and you need to test whether injection-order-dependent bias remains.

## When NOT to use

- The data are integrated peak areas rather than final quantified concentrations — QuantyFey performs quantification first and does not provide peak integration capabilities itself
- The goal is to compare correction methods on accuracy metrics (mean bias, limits of agreement, Passing-Bablok regression) rather than to detect residual injection-order drift
- The input comes from untargeted/untreated MS data where no external-calibration quantification table and no defined injection order exist

## Inputs

- Quantified concentration table per compound per drift correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) exported from QuantyFey
- Injection order (position) of the LC-MS/MS sequence
- Raw, uncorrected concentrations for the baseline drift assessment

## Outputs

- Table of Spearman correlation coefficients and p-values per compound per correction strategy
- List of compound/strategy pairs flagged with significant residual drift (p <= 0.05)
- Demonstration of IS-analogue mismatch bias for 4-hydroxy-proline and methionine sulfoxide (increased values at sequence end)

## How to apply

Load the quantified concentration table produced under each correction strategy together with the injection order of the sequence (e.g., the porcine plasma LC-MS/MS dataset from QuantyFey). For each compound and each strategy, compute the Spearman rank correlation between injection position and determined concentration with R's stats::cor.test(method="spearman"), and repeat the test on raw, uncorrected concentrations to establish the baseline residual drift. For compounds with suspected IS-analogue mismatch (e.g., 4-hydroxy-proline and methionine sulfoxide vs analogues proline and methionine), additionally compute the correlation under IS correction to reproduce the end-of-sequence upward bias. Tabulate correlation coefficients and p-values per compound per strategy and flag strategies with significant residual drift; a strategy is judged drift-free when its Spearman correlation is non-significant (p > 0.05), consistent with the paper's finding that both QC-based and IS-based correction significantly reduced drift effects while weighted/custom bracketing showed compound-dependent performance.

## Related tools

- **QuantyFey** (Produces the quantified concentration tables under each drift correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) that feed the Spearman drift test) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime used to run stats::cor.test with method="spearman" on injection position vs determined concentration; QuantyFey itself is implemented in R (v4.2.0))
- **MultiQuant 3.0.3** (Upstream vendor software in which peak integration and manual review were performed before peak areas and RTs were exported for quantification)

## Examples

```
cor.test(x = injection_order, y = conc_table[, cmp], method = "spearman")
```

## Evaluation signals

- A strategy is judged drift-free when its Spearman correlation between injection position and concentration is non-significant (p > 0.05)
- The raw, uncorrected concentrations should show the baseline significant residual drift against which corrected strategies are compared
- For 4-hydroxy-proline and methionine sulfoxide under IS correction, the test should reproduce significant drift with increased values at the end of the sequence, matching the analogue-mismatch finding
- QC-based and IS-based correction are expected to show significantly reduced drift effects relative to raw data, while weighted/custom bracketing may show compound-dependent residual drift
- Correlation coefficients and p-values must be tabulated for every compound/strategy combination so flagging is reproducible

## Limitations

- Weighted bracketing performed worse than loess-based approaches in simulation and showed variable performance across compounds, so a non-significant Spearman result under one strategy does not generalize across all compounds
- IS correction with an unsuitable IS (e.g., 4-hydroxy-proline paired with proline, methionine sulfoxide paired with methionine) can itself introduce bias — differing trends between analyte and analogue led to increased values at the end of the sequence
- QuantyFey is compatible with Windows operating systems only (standalone version); Linux/Mac require the Apptainer version or running from R, and the app works only on already integrated MS data without integration capabilities

## Evidence

- [methods] compute Spearman rank correlation between injection position and determined concentration in R (stats::cor.test with method="spearman"): [evidence span withheld — anchor status: not_found]
- [abstract] for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases, leading to increased values at the: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases, leading to increased values at the"
- [abstract] Both QC-based and IS-based correction significantly reduced drift effects.: "Both QC-based and IS-based correction significantly reduced drift effects."
- [abstract] Custom- or weighted bracketing methods also improved quantification accuracy but demonstrated variable performance across compounds.: "Custom- or weighted bracketing methods also improved quantification accuracy but demonstrated variable performance across compounds."
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
