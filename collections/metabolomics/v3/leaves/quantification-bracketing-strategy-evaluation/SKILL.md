---
name: quantification-bracketing-strategy-evaluation
description: Use when targeted LC-MS/MS peak areas show intensity drift and you must
  decide which drift correction or bracketing strategy to apply, by comparing concentrations
  across IS correction, QC-based, custom, and weighted bracketing.
when_to_use_negative:
- Peak integration has not yet been performed — QuantyFey works as additional software
  on already integrated mass spectrometry data and provides no integration capabilities
- Only a single correction strategy was applied, leaving no cross-method concentrations
  to correlate or compare
- The dataset is not targeted LC-MS/MS external-calibration data compatible with QuantyFey's
  Windows-only standalone version (or an Apptainer/Linux setup)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: QuantyFey
  role: Shiny application used to compute concentrations under each drift correction/bracketing
    strategy and export the results table for comparison
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Used to compute pairwise Pearson correlation coefficients between the method
    columns and assemble the correlation matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:7ce15340de0afdf92f08431d73f3ac18fafbe7d14b3baa461e4c37a8d51bb818
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QuantyFey
  - R 4.2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# quantification-bracketing-strategy-evaluation

## Summary

Comparative evaluation of LC-MS/MS drift correction and quantification strategies (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) using mean bias, limits of agreement (LoA), Passing-Bablok regression, and Pearson/Spearman correlation of determined concentrations.

## When to use

Use when targeted LC-MS/MS peak areas show intensity drift and you must decide which drift correction or bracketing strategy to apply, by comparing concentrations across IS correction, QC-based, custom, and weighted bracketing.

## When NOT to use

- Peak integration has not yet been performed — QuantyFey works as additional software on already integrated mass spectrometry data and provides no integration capabilities
- Only a single correction strategy was applied, leaving no cross-method concentrations to correlate or compare
- The dataset is not targeted LC-MS/MS external-calibration data compatible with QuantyFey's Windows-only standalone version (or an Apptainer/Linux setup)

## Inputs

- Concentration results table exported from QuantyFey with one column per correction strategy (default bracketing, QC-based drift correction, custom bracketing, weighted bracketing)
- Peak areas and retention times exported from vendor software (MultiQuant 3.0.3) after peak integration and manual review
- Calibration and QC sample annotations within the injection sequence

## Outputs

- Four-by-four pairwise Pearson correlation matrix of concentrations across correction strategies
- Mean bias and limits of agreement (LoA) between compared methods
- Strategy recommendation per compound (with confidence on compound-specific suitability)

## How to apply

Process each compound with up to four approaches (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) in QuantyFey, exporting a concentration table (e.g., 25 porcine plasma compounds) with one column per strategy. Compute mean bias as the average of relative differences between compared methods and derive limits of agreement (LoA); optionally run Passing-Bablok regression. Compute pairwise Pearson (or Spearman) correlation across the method columns with cor(method='pearson') in R 4.2.0 to build a four-by-four matrix. Validate against the paper's pattern: QC-based, custom, and weighted bracketing mutually correlate at Pearson r > 0.9, while default bracketing and IS correction correlate lower; also check whether IS-corrected concentrations sit consistently lower (about a factor of two, indicating matrix enhancement). Treat strategy selection as compound-specific, since unsuitable IS choices (e.g., 4-hydroxy-proline and methionine sulfoxide) introduced biases.

## Related tools

- **QuantyFey** (Shiny application used to compute concentrations under each drift correction/bracketing strategy and export the results table for comparison) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Used to compute pairwise Pearson correlation coefficients between the method columns and assemble the correlation matrix)

## Examples

```
cor(conc[, c("default_bracketing","qc_drift","custom_bracketing","weighted_bracketing")], method='pearson')
```

## Evaluation signals

- The three non-IS strategies (QC-based drift correction, custom bracketing, weighted bracketing) mutually correlate at Pearson r > 0.9 across the 25 compounds
- Default bracketing and IS correction columns show lower Pearson correlation with the other quantification methods
- IS-corrected concentrations are consistently lower than other methods by approximately a factor of two (expected under matrix enhancement)
- Mean bias equals the average of relative differences between compared methods and LoA brackets the observed method differences
- Weighted bracketing results are highly similar to QC-based drift correction despite linear weighting versus non-linear loess

## Limitations

- Custom and weighted bracketing improved quantification accuracy but showed variable performance across compounds
- Unsuitable IS selection (e.g., 4-hydroxy-proline and methionine sulfoxide, whose analogues proline and methionine differed in drift trends) introduced biases — strategy choice is compound-specific
- QuantyFey is compatible with Windows operating systems only for the standalone version; the Apptainer setup is complicated and slow on macOS

## Evidence

- [abstract] Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [abstract] mean bias and LoA definition: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [abstract] weighted bracketing vs QC-based similarity: "Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the"
- [readme] QuantyFey purpose: "It is specifically designed to address **intensity drifts** in datasets, offering multiple **correction strategies** to ensure accurate quantification."
