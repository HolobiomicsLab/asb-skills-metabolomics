---
name: drift-correction-strategy-comparison
description: Use when a targeted LC-MS/MS sequence shows intensity drift and multiple
  drift-correction strategies (IS, QC-based loess, custom/weighted bracketing) have
  produced concentration tables that must be evaluated for agreement and residual
  drift.
when_to_use_negative:
- The dataset has not yet been quantified under multiple correction strategies — first
  run QuantyFey quantification under each strategy before comparing outputs
- Raw chromatograms need peak integration — QuantyFey works as additional software
  to already integrated mass spectrometry data and does not provide integration capabilities
- The goal is calibration model selection rather than correction-strategy agreement
  — use QuantyFey's automatic/interactive regression optimization module instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QuantyFey
  role: Shiny application that produces the concentration tables under the four drift-correction
    strategies being compared
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Statistical environment for Passing-Bablok regression and Spearman correlation
    testing (stats::cor.test)
- name: Shiny 1.10.0
  role: Framework hosting the QuantyFey app used to generate the strategy-specific
    concentration tables
- name: MultiQuant 3.0.3
  role: Vendor software used upstream for peak integration and export of peak areas
    and retention times
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:33d8ed12cf74805795c50005e082ae15223417fd7e0510a0b170f6e2a48121fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  - Shiny
  - Shiny 1.10.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# drift-correction-strategy-comparison

## Summary

Compare up to four drift-correction strategies (IS correction, QC-based loess drift correction, custom bracketing, weighted bracketing) applied to targeted LC-MS/MS quantification, using mean bias, limits of agreement, Passing-Bablok regression, and Spearman correlation of concentration vs injection position.

## When to use

Use when a targeted LC-MS/MS sequence shows intensity drift and multiple drift-correction strategies (IS, QC-based loess, custom/weighted bracketing) have produced concentration tables that must be evaluated for agreement and residual drift.

## When NOT to use

- The dataset has not yet been quantified under multiple correction strategies — first run QuantyFey quantification under each strategy before comparing outputs
- Raw chromatograms need peak integration — QuantyFey works as additional software to already integrated mass spectrometry data and does not provide integration capabilities
- The goal is calibration model selection rather than correction-strategy agreement — use QuantyFey's automatic/interactive regression optimization module instead

## Inputs

- Per-compound, per-strategy concentration tables exported from QuantyFey (IS correction, QC-based loess drift correction with span 0.75, custom bracketing, weighted bracketing, plus raw uncorrected concentrations)
- Injection order / sequence position of samples in the porcine plasma LC-MS/MS run
- Peak areas and retention times exported from MultiQuant 3.0.3

## Outputs

- passing_bablok_per_compound.csv (per-compound slopes and intercepts for all six pairwise strategy comparisons)
- passing_bablok_summary.csv (aggregated slope/intercept summary per strategy comparison)
- Spearman correlation coefficients and p-values per compound per strategy, flagging strategies with significant residual drift
- Mean bias and limits of agreement (LoA) per method comparison

## How to apply

Load the per-compound concentration tables (e.g., 25 porcine plasma compounds, 1–1000 µg/L range) produced under each of the four correction strategies together with the injection order. For each compound, form all six pairwise strategy comparisons and fit Passing-Bablok regression (pairwise-median slope estimator); slopes close to 1 indicate agreement, while deviations (as seen against IS correction) reveal matrix effects and overestimation when correction is improper. Compute mean bias as the average of relative differences between compared methods, then derive limits of agreement (LoA). For residual-drift assessment, run Spearman rank correlation (stats::cor.test, method='spearman') between injection position and concentration per compound per strategy, plus raw uncorrected concentrations as baseline; a strategy is drift-free when correlation is non-significant (p > 0.05). Flag compound-specific failures such as IS-analogue mismatch (4-hydroxy-proline vs proline, methionine sulfoxide vs methionine) that reintroduce end-of-sequence bias.

## Related tools

- **QuantyFey** (Shiny application that produces the concentration tables under the four drift-correction strategies being compared) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Statistical environment for Passing-Bablok regression and Spearman correlation testing (stats::cor.test))
- **Shiny 1.10.0** (Framework hosting the QuantyFey app used to generate the strategy-specific concentration tables)
- **MultiQuant 3.0.3** (Vendor software used upstream for peak integration and export of peak areas and retention times)

## Examples

```
cor.test(conc, injection_order, method="spearman")  # per compound per strategy in R 4.2.0
```

## Evaluation signals

- Passing-Bablok slopes should be generally close to 1 for all compound comparisons except those involving IS correction, reproducing the paper's matrix-enhancement finding
- IS-corrected concentrations should be consistently lower by approximately a factor of two compared to other methods
- QC-based drift correction and weighted bracketing should yield highly similar concentration results despite linear vs non-linear (loess) weighting
- A strategy is judged drift-free when Spearman correlation between injection position and concentration is non-significant (p > 0.05); raw uncorrected concentrations serve as the significant-drift baseline
- For 4-hydroxy-proline and methionine sulfoxide under IS correction, positive Spearman correlation should reproduce the analogue-mismatch bias increasing values at sequence end

## Limitations

- IS correction is unsuitable for compounds with analogue mismatch (4-hydroxy-proline, methionine sulfoxide), introducing end-of-sequence biases; strategy performance is compound-dependent
- Custom and weighted bracketing improve accuracy but show variable performance across compounds, and weighted bracketing performed worse than loess-based approaches in simulation
- QuantyFey is compatible with Windows only (standalone version); Linux/macOS require the Apptainer version or local R launch

## Evidence

- [abstract] the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [abstract] Both QC-based and IS-based correction significantly reduced drift effects.: "Both QC-based and IS-based correction significantly reduced drift effects."
- [results] Passing-Bablok analysis showed slopes generally close to 1 for all compounds across all method comparisons except when compared to IS correction: [evidence span withheld — anchor status: not_found]
- [abstract] for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases"
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
