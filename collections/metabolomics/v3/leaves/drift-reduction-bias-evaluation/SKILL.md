---
name: drift-reduction-bias-evaluation
description: Use when comparing drift-correction strategies (IS correction, QC-based
  correction, bracketing) on a targeted LC-MS/MS sequence to quantify residual bias
  and confirm drift reduction objectively.
when_to_use_negative:
- The data are raw vendor peak tables that have not yet been drift-corrected and quantified
  via external calibration; this skill evaluates corrected concentrations, not raw
  intensities.
- The analysis target is chromatographic peak integration or calibration-curve fitting
  itself; QuantyFey explicitly does not provide integration capabilities and this
  evaluation assumes integrated peak areas are already available.
- Comparing methods on unstructured/unordered samples where injection position is
  unknown, since drift evaluation relies on correlating concentration against injection
  order.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QuantyFey
  role: Generates the drift-corrected, externally calibrated concentrations from integrated
    peak areas/RTs that are then compared using the bias/LoA/Passing-Bablok/correlation
    evaluation metrics.
  repo: https://github.com/CDLMarkus/QuantyFey
- name: R 4.2.0
  role: Implementation environment in which QuantyFey and its drift-correction strategies
    run, providing the statistical functions for bias, LoA, Passing-Bablok regression,
    and Pearson/Spearman correlation calculations.
  repo: https://github.com/CDLMarkus/QuantyFey
- name: MultiQuant 3.0.3
  role: Performs peak integration and manual chromatographic review; exports peak
    areas and retention times that feed the upstream QuantyFey quantification whose
    outputs this skill evaluates.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2025.344571
    title: 'QuantyFey: An open-source tool for targeted LC-MS quantification with
      integrated drift correction'
  source_content_hash: sha256:c828f818e2f28ea5d2803025ae14e4025e641004cccecf486774b914cffb9a59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CDLMarkus/QuantyFey
  tools:
  - MultiQuant
  - MultiQuant 3.0.3
  - QuantyFey
  - R 4.2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# drift-reduction-bias-evaluation

## Summary

Evaluation methodology from the QuantyFey paper for assessing whether LC-MS/MS intensity drift has been reduced: computing mean bias, limits of agreement, Passing-Bablok regression, and Pearson/Spearman correlations of corrected vs. uncorrected concentrations against injection position.

## When to use

Use when comparing drift-correction strategies (IS correction, QC-based correction, bracketing) on a targeted LC-MS/MS sequence to quantify residual bias and confirm drift reduction objectively.

## When NOT to use

- The data are raw vendor peak tables that have not yet been drift-corrected and quantified via external calibration; this skill evaluates corrected concentrations, not raw intensities.
- The analysis target is chromatographic peak integration or calibration-curve fitting itself; QuantyFey explicitly does not provide integration capabilities and this evaluation assumes integrated peak areas are already available.
- Comparing methods on unstructured/unordered samples where injection position is unknown, since drift evaluation relies on correlating concentration against injection order.

## Inputs

- Corrected and uncorrected concentration matrices from a targeted LC-MS/MS sequence (compounds x samples, with injection order retained)
- Peak areas and retention times exported from vendor software (e.g. MultiQuant) for each compound across the injection sequence
- Drift-correction outputs per compound: IS-corrected, QC-based corrected, custom-bracketing, and weighted-bracketing concentrations

## Outputs

- Mean bias (average relative difference between compared methods) per compound
- Limits of agreement (LoA) per compound
- Passing-Bablok regression comparison results
- Pearson correlation coefficient of concentration vs. injection position per compound
- Spearman correlation coefficient of concentration vs. injection position per compound

## How to apply

After applying candidate drift-correction strategies to a targeted LC-MS/MS dataset (e.g. the porcine plasma dataset with 25 compounds processed with IS correction, QC-based correction, custom bracketing, and weighted bracketing), compute the mean bias as the average of relative differences between compared methods. Derive limits of agreement (LoA) from those relative differences to capture the spread of method agreement. Characterize residual drift by regressing concentration against injection position and reporting both Pearson and Spearman correlation coefficients, for each correction strategy and compound. Use Passing-Bablok regression to compare method pairs, which is robust to non-normal concentration distributions. Interpret compound-specific results carefully: the paper found that unsuitable internal-standard selection (e.g. 4-hydroxy-proline and methionine sulfoxide paired with proline and methionine analogues with differing trends) introduced biases that the evaluation framework exposed.

## Related tools

- **QuantyFey** (Generates the drift-corrected, externally calibrated concentrations from integrated peak areas/RTs that are then compared using the bias/LoA/Passing-Bablok/correlation evaluation metrics.) — https://github.com/CDLMarkus/QuantyFey
- **R 4.2.0** (Implementation environment in which QuantyFey and its drift-correction strategies run, providing the statistical functions for bias, LoA, Passing-Bablok regression, and Pearson/Spearman correlation calculations.) — https://github.com/CDLMarkus/QuantyFey
- **MultiQuant 3.0.3** (Performs peak integration and manual chromatographic review; exports peak areas and retention times that feed the upstream QuantyFey quantification whose outputs this skill evaluates.)

## Examples

```
Rscript -e 'data <- read.csv("corrected_concentrations.csv"); bias <- mean((data$corrected - data$uncorrected)/data$uncorrected); cor.test(seq_along(data$corrected), data$corrected, method="pearson")'
```

## Evaluation signals

- For each compound, the mean bias between a corrected method and the uncorrected reference should be near zero if drift was successfully reduced; large persistent mean bias indicates an inappropriate correction (e.g. IS-matrix mismatch like the ~factor-of-two lowering seen for IS-corrected results).
- Limits of agreement should tighten relative to uncorrected-vs-uncorrected self-comparison when a correction genuinely reduces drift; wide LoA signals residual noise.
- Pearson and Spearman correlation of concentration vs. injection position should drop toward zero after effective drift correction; a strong remaining monotonic trend means drift persists.
- Passing-Bablok regression slopes should be consistent with unit agreement between methods expected to be equivalent (e.g. QC-based correction and weighted bracketing, which the paper found yielded highly similar results).
- Cross-check that the compound-specific corrections with known IS problems (4-hydroxy-proline, methionine sulfoxide) show increased bias at the sequence end, matching the paper's reported behaviour.

## Limitations

- IS correction can be systematically biased: IS-corrected concentrations were consistently lower by approximately a factor of two compared to other methods in the porcine plasma dataset, suggesting matrix enhancement upon LC-MS/MS analysis.
- Compound-specific IS analogue mismatch (4-hydroxy-proline vs. proline; methionine sulfoxide vs. methionine) introduced biases and increased values at the end of the sequence, so a single global bias metric may mask compound-level problems.
- Weighted bracketing showed variable performance across compounds, so averaged bias/LoA metrics alone can obscure compound-specific adequacy.
- QuantyFey is compatible with Windows operating systems only (or via the Apptainer route), limiting direct reproduction of the evaluation environment on other platforms.

## Evidence

- [abstract] the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined: "the mean bias was calculated as the average of relative differences between compared methods. Limits of agreement (LoA) were then determined"
- [intro] QuantyFey is a Shiny application for the visualization, analysis, and quantification of mass spectrometry (MS) data using external calibration: "**QuantyFey** is a Shiny application for the **visualization, analysis, and quantification** of **mass spectrometry (MS) data** using **external calibration**"
- [abstract] Both QC-based and IS-based correction significantly reduced drift effects.: "Both QC-based and IS-based correction significantly reduced drift effects."
- [abstract] Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the: "Notable, QC-based drift correction and weighted bracketing yielded highly similar results, which is especially interesting given that weighted bracketing applied a linear weight change over the"
- [abstract] for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases, leading to increased values at the: "for compounds like 4-hydroxy-proline and methionine sulfoxide, differing trends between the analytes and the analogues (proline and methionine) introduced biases, leading to increased values at the"
- [readme] This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities: "This app works as an additional software to already integrated mass spectrometry data, and does not provide any integration cabailities"
