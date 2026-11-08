---
name: targeted-ms-quantification-qc-evaluation
description: Use when a targeted LC-MS/MS sequence has been quantified under one or
  more drift-correction strategies (IS correction, QC-based drift correction, custom
  bracketing, weighted bracketing) and you must verify whether residual drift remains,
  per compound, before reporting concentrations.
when_to_use_negative:
- 'Input is still un-integrated chromatographic data: QuantyFey works as additional
  software to already integrated mass spectrometry data and provides no integration
  capabilities'
- The dataset lacks injection-order metadata or calibration/QC block structure, so
  position-vs-concentration testing and bracketing strategies are undefined
- Only a single correction strategy was applied and no raw uncorrected baseline exists,
  so the drift-reduction effect cannot be assessed comparatively
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QuantyFey
  role: Shiny application that generates the quantified concentration tables under
    each drift-correction strategy (IS correction, QC-based drift correction, custom
    bracketing, weighted bracketing) that are then evaluated for residual drift
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Runtime in which QuantyFey is implemented and in which stats::cor.test(method="spearman")
    drift tests are performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:7a636c3088ed2a7d285a4f251bf5c6c359bf7cf2dcc8986fa93bcad58e1f353e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Targeted MS Quantification QC Evaluation

## Summary

Quality-control evaluation of targeted LC-MS/MS quantification outputs, using Spearman rank correlation of determined concentration vs injection position (plus mean bias, limits of agreement, Passing-Bablok regression, and Pearson correlation) to detect residual intensity drift across correction strategies.

## When to use

Use when a targeted LC-MS/MS sequence has been quantified under one or more drift-correction strategies (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) and you must verify whether residual drift remains, per compound, before reporting concentrations.

## When NOT to use

- Input is still un-integrated chromatographic data: QuantyFey works as additional software to already integrated mass spectrometry data and provides no integration capabilities
- The dataset lacks injection-order metadata or calibration/QC block structure, so position-vs-concentration testing and bracketing strategies are undefined
- Only a single correction strategy was applied and no raw uncorrected baseline exists, so the drift-reduction effect cannot be assessed comparatively

## Inputs

- Quantified concentration table per correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) exported from QuantyFey
- Injection order of the targeted LC-MS/MS sequence
- Raw, uncorrected concentrations (baseline drift reference)
- Peak areas and retention times exported from vendor integration software (e.g., MultiQuant 3.0.3)

## Outputs

- Per-compound, per-strategy table of Spearman correlation coefficients and p-values (concentration vs injection position)
- Flagged list of compounds/strategies with significant residual drift
- Cross-method comparison metrics: mean bias, limits of agreement (LoA), Passing-Bablok regression, Pearson correlation
- Identification of analogue-mismatch compounds (e.g., 4-hydroxy-proline, methionine sulfoxide) with persistent post-correction drift

## How to apply

Load the quantified concentration table produced under each correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) together with the injection order of the sequence. For each compound and each strategy, compute Spearman rank correlation between injection position and determined concentration in R using stats::cor.test with method="spearman"; repeat the test on raw, uncorrected concentrations to establish the baseline residual drift. Judge a strategy drift-free when its Spearman correlation is non-significant (p > 0.05). For cross-method comparison, additionally compute mean bias (average of relative differences between compared methods) and limits of agreement (LoA), Passing-Bablok regression, and Pearson correlation. Explicitly inspect IS-corrected results for compounds whose internal-standard analogue mismatched the analyte (e.g., 4-hydroxy-proline vs proline, methionine sulfoxide vs methionine): differing analyte/analogue drift trends reintroduce bias that increases values at the sequence end even after correction.

## Related tools

- **QuantyFey** (Shiny application that generates the quantified concentration tables under each drift-correction strategy (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) that are then evaluated for residual drift) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Runtime in which QuantyFey is implemented and in which stats::cor.test(method="spearman") drift tests are performed)

## Examples

```
cor.test(injection_order, concentration, method="spearman")
```

## Evaluation signals

- A correction strategy is judged drift-free when its Spearman correlation between injection position and concentration is non-significant (p > 0.05), while the raw uncorrected baseline should show significant correlation
- Both QC-based and IS-based correction should significantly reduce drift effects, whereas custom and weighted bracketing show compound-dependent (variable) performance
- IS-corrected concentrations should come out consistently lower — by approximately a factor of two — than other methods, indicating matrix enhancement; a deviation suggests a processing error
- For 4-hydroxy-proline and methionine sulfoxide under IS correction, expect residual bias (increased values at the sequence end) due to analyte/analogue trend mismatch with proline and methionine — flagging these as failures is expected, not a bug
- QC-based drift correction and weighted bracketing should yield highly similar results despite linear weighting vs non-linear loess, serving as a cross-check on the comparison pipeline

## Limitations

- Unsuitable IS selection introduces compound-specific biases: analyte/analogue trend mismatch (e.g., 4-hydroxy-proline vs proline) persists after IS correction
- Custom and weighted bracketing improved quantification accuracy but showed variable performance across compounds, so no single strategy is universally drift-free
- QuantyFey's standalone version is compatible with Windows only; the apptainer version is (more) complicated on macOS and slow
- QuantyFey does not provide peak-integration capabilities — data must be integrated beforehand (e.g., in MultiQuant 3.0.3)

## Evidence

- [abstract] Both QC-based and IS-based drift correction significantly reduced drift effects in a targeted LC-MS/MS dataset of porcine plasma.: "Both QC-based and IS-based correction significantly reduced drift effects."
- [abstract] Cross-method evaluation metrics.: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [abstract] Analogue-mismatch residual drift.: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases"
- [readme] QuantyFey scope: quantification of already-integrated data.: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
