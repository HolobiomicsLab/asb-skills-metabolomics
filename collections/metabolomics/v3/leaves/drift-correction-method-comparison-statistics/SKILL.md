---
name: drift-correction-method-comparison-statistics
description: Use when multiple drift correction or quantification strategies have
  been applied to the same targeted LC-MS/MS dataset and you need to quantify their
  agreement (e.g., test whether non-IS strategies mutually correlate at Pearson r
  > 0.9).
when_to_use_negative:
- The MS data have not yet been peak-integrated or reviewed — QuantyFey works only
  on already integrated data and provides no integration capabilities
- The goal is to correct drift itself rather than compare the outputs of correction
  strategies (apply QuantyFey's IS correction, QC-based drift correction, custom bracketing,
  or weighted bracketing modules first)
- Only a single correction strategy was applied to the dataset — with one method column
  there is nothing to compare
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QuantyFey
  role: Shiny application that computes concentrations for each compound under up
    to four drift correction strategies, producing the results table compared statistically
  repo: https://github.com/CDLMarkus/QuantyFey/releases/
- name: R 4.2.0
  role: Statistics environment used to compute pairwise Pearson correlations (cor(method='pearson')),
    mean bias, LoA, Passing-Bablok regression, and Spearman correlations
- name: MultiQuant 3.0.3
  role: Upstream vendor software for peak integration and manual review of chromatographic
    data whose exported peak areas and RTs feed the comparison workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:08cd904571ede7b6e0caaeb85e2b4b5126b4fc11e9e0f9c62900777f5b0d6dc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# drift-correction-method-comparison-statistics

## Summary

Statistical comparison of LC-MS/MS drift correction strategies (IS correction, QC-based drift correction, custom bracketing, weighted bracketing) using mean bias, limits of agreement, Passing-Bablok regression, and Pearson/Spearman correlation of computed concentrations.

## When to use

Use when multiple drift correction or quantification strategies have been applied to the same targeted LC-MS/MS dataset and you need to quantify their agreement (e.g., test whether non-IS strategies mutually correlate at Pearson r > 0.9).

## When NOT to use

- The MS data have not yet been peak-integrated or reviewed — QuantyFey works only on already integrated data and provides no integration capabilities
- The goal is to correct drift itself rather than compare the outputs of correction strategies (apply QuantyFey's IS correction, QC-based drift correction, custom bracketing, or weighted bracketing modules first)
- Only a single correction strategy was applied to the dataset — with one method column there is nothing to compare

## Inputs

- QuantyFey concentration results table with one column per drift correction strategy (default bracketing, QC-based drift correction, custom bracketing, weighted bracketing)
- Peak areas and retention times exported from vendor software (MultiQuant 3.0.3) for targeted LC-MS/MS data

## Outputs

- Pairwise Pearson correlation matrix (e.g., 4x4) of concentrations across correction strategies
- Mean bias and limits of agreement (LoA) between compared methods
- Passing-Bablok regression estimates per method pair
- Spearman correlation of concentration vs injection position per strategy

## How to apply

Export the concentration results table from QuantyFey, which contains one column per strategy (default bracketing, QC-based drift correction, custom bracketing, weighted bracketing) per compound — in the paper, 25 porcine plasma compounds processed with up to four approaches. In R 4.2.0, compute pairwise Pearson correlation coefficients between the method columns using cor(method='pearson') and assemble the full correlation matrix; QC-based, custom, and weighted bracketing are expected to mutually correlate at r > 0.9, while default bracketing and IS correction correlate lower with the other methods. Complement this with mean bias (average of relative differences between compared methods), limits of agreement (LoA), Passing-Bablok regression, and Spearman correlation of concentration versus injection position to capture drift remaining in each strategy. Interpret IS-corrected results in light of expected systematic offsets — here IS correction was consistently lower by roughly a factor of two, suggesting matrix enhancement — rather than treating low correlation alone as failure.

## Related tools

- **QuantyFey** (Shiny application that computes concentrations for each compound under up to four drift correction strategies, producing the results table compared statistically) — https://github.com/CDLMarkus/QuantyFey/releases/
- **R 4.2.0** (Statistics environment used to compute pairwise Pearson correlations (cor(method='pearson')), mean bias, LoA, Passing-Bablok regression, and Spearman correlations)
- **MultiQuant 3.0.3** (Upstream vendor software for peak integration and manual review of chromatographic data whose exported peak areas and RTs feed the comparison workflow)

## Examples

```
r <- cor(conc[, c("default_bracketing","QC_drift_correction","custom_bracketing","weighted_bracketing")], method="pearson")
```

## Evaluation signals

- The three non-IS correction strategies (QC-based drift correction, custom bracketing, weighted bracketing) mutually correlate at Pearson r > 0.9
- Default bracketing and IS correction columns show lower correlation with the other quantification methods
- Correlation matrix is square (n strategies x n strategies), symmetric, with diagonal equal to 1
- IS-corrected concentrations are consistently lower — approximately a factor of two — than other methods, consistent with matrix enhancement
- Weighted bracketing and QC-based drift correction yield highly similar results despite linear weighting vs non-linear loess

## Limitations

- Unsuitable IS selection (e.g., 4-hydroxy-proline and methionine sulfoxide) introduced biases due to differing trends between analytes and analogues, so IS-correlation results are compound-specific
- Custom and weighted bracketing improved quantification accuracy but showed variable performance across compounds
- QuantyFey (and its standalone version) is compatible with Windows operating systems only; other platforms require the Apptainer version

## Evidence

- [abstract] Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing: "Each compound was processed with up to four approaches: IS correction, QC-based drift correction, custom bracketing, and weighted bracketing"
- [abstract] the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [abstract] IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods.: "IS corrected results were consistently lower – by approximately a factor of two – compared to those from the other methods."
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
