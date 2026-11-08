---
name: is-analogue-mismatch-bias-assessment
description: Use when a targeted LC-MS/MS sequence was quantified with IS correction
  and you must check whether the chosen IS analogue actually tracks the analyte's
  drift — e.g. 4-hydroxy-proline vs proline, methionine sulfoxide vs methionine.
when_to_use_negative:
- Peak integration has not yet been performed — QuantyFey 'works as an additional
  software to already integrated mass spectrometry data, and does not provide any
  integration capabilities'
- You need absolute accuracy benchmarking between methods rather than residual-drift
  assessment — use mean bias, limits of agreement, and Passing-Bablok regression instead
- The sequence was processed with QC-based or bracketing strategies only and no IS
  assignment exists, so no analogue-mismatch hypothesis can be tested
tools:
- name: QuantyFey
  role: Produces the per-strategy quantified concentration tables (IS correction,
    QC-based drift correction, custom bracketing, weighted bracketing) that feed the
    Spearman drift assessment
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runs stats::cor.test with method="spearman" between injection position and
    determined concentration for each compound and strategy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:e7bf8a55bd0500770915db3c2f459dcf2d01a64ce4ecc912776a7f48e163b49c
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# IS-analogue mismatch bias assessment

## Summary

Detects residual concentration drift caused by internal-standard analogue mismatch: when an analyte's drift trend differs from its assigned analogue's trend, IS correction introduces bias (e.g. elevated values at sequence end). Assessed by Spearman correlation of concentration vs injection position.

## When to use

Use when a targeted LC-MS/MS sequence was quantified with IS correction and you must check whether the chosen IS analogue actually tracks the analyte's drift — e.g. 4-hydroxy-proline vs proline, methionine sulfoxide vs methionine.

## When NOT to use

- Peak integration has not yet been performed — QuantyFey 'works as an additional software to already integrated mass spectrometry data, and does not provide any integration capabilities'
- You need absolute accuracy benchmarking between methods rather than residual-drift assessment — use mean bias, limits of agreement, and Passing-Bablok regression instead
- The sequence was processed with QC-based or bracketing strategies only and no IS assignment exists, so no analogue-mismatch hypothesis can be tested

## Inputs

- Quantified concentration table per correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) exported from QuantyFey
- Injection order (sequence position) of the porcine plasma LC-MS/MS run
- Raw, uncorrected concentrations for baseline drift comparison
- Compound-to-IS-analogue assignment (e.g. 4-hydroxy-proline -> proline, methionine sulfoxide -> methionine)

## Outputs

- Table of Spearman correlation coefficients and p-values per compound per correction strategy (plus raw baseline)
- Flagged compounds with significant residual drift (p <= 0.05) under each strategy
- Identification of analogue-mismatch cases where analyte and analogue trends diverge, biasing values upward at sequence end

## How to apply

Load the quantified concentration table produced under each correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) together with the injection order of the sequence, and also the raw, uncorrected concentrations to establish the baseline residual drift. For each compound and each strategy, compute the Spearman rank correlation between injection position and determined concentration in R with stats::cor.test(method="spearman"). A strategy is judged drift-free when its Spearman correlation is non-significant (p > 0.05); significant positive rho indicates residual drift toward higher values at sequence end. For compounds suspected of analogue mismatch (4-hydroxy-proline and methionine sulfoxide), compute the Spearman correlation specifically under IS correction and compare the analyte's trend with that of its analogue (proline and methionine): differing trends between analyte and analogue under IS correction reproduce the mismatch bias that increases values at the end of the sequence. Tabulate coefficients and p-values per compound per strategy to flag strategies with significant residual drift and compounds needing an alternative IS or strategy.

## Related tools

- **QuantyFey** (Produces the per-strategy quantified concentration tables (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) that feed the Spearman drift assessment) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runs stats::cor.test with method="spearman" between injection position and determined concentration for each compound and strategy)

## Examples

```
cor.test(injection_position, concentration, method="spearman")  # per compound, per correction strategy in R
```

## Evaluation signals

- Spearman p > 0.05 for concentration vs injection position under the chosen strategy (drift-free criterion)
- Significant positive Spearman rho persists for 4-hydroxy-proline and methionine sulfoxide under IS correction, matching the reported analogue-mismatch bias (increased values at sequence end)
- Raw uncorrected concentrations show significant baseline drift that is reduced by IS- and QC-based correction
- IS-corrected concentrations are consistently lower by approximately a factor of two than other methods (matrix enhancement), a sanity check when comparing strategies
- QC-based drift correction and weighted bracketing yield highly similar results, providing an expected cross-method agreement pattern

## Limitations

- IS correction is unsuitable when the analogue's drift trend differs from the analyte's, as for 4-hydroxy-proline and methionine sulfoxide; bracketing strategies showed compound-dependent performance
- IS-corrected results were consistently lower by approximately a factor of two, suggesting matrix enhancement upon LC-MS/MS analysis
- QuantyFey is compatible with Windows only (standalone); Linux/macOS require Apptainer or launch from R 4.2.x/4.5.x

## Evidence

- [abstract] for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases"
- [abstract] Both QC-based and IS-based correction significantly reduced drift effects.: "Both QC-based and IS-based correction significantly reduced drift effects."
- [abstract] Evaluation of drift correction methods via mean bias, limits of agreement, Passing-Bablok regression, Pearson correlation, and Spearman correlation of concentration vs injection position: "the mean bias was calculated as the average of relative differences between compared methods"
- [abstract] IS corrected results were consistently lower by approximately a factor of two compared to other methods, suggesting matrix enhancement.: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods. This suggests matrix enhancement"
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
