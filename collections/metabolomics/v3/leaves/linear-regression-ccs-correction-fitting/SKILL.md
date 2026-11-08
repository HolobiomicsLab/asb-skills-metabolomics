---
name: linear-regression-ccs-correction-fitting
description: Use when TIM-MS or DTIM-MS lipidomics CCS values show class-specific
  bias against a DTCCSN2 reference library and at least 3 U13C-labeled lipids per
  lipid class-adduct combination are available for regression.
when_to_use_negative:
- Fewer than 3 U13C-labeled lipids exist for a lipid class-adduct combination, since
  linear regression requires a minimum of 3 lipids per function (correction restricted
  to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG)
- Input measured values are already corrected CCS values — this skill fits functions
  from uncorrected measured CCS vs library pairs and should not be reapplied to corrected
  output
tools:
- name: MobiLipid
  role: R Markdown tool that resamples U13C-labeled lipids, fits the linear correction
    functions, applies them, and reports post-correction bias
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: R Markdown
  role: Execution format wrapping the CCS correction workflow (MobiLipid_CCS-bias-calculation_CCS-correction.Rmd)
  repo: https://github.com/FelinaHildebrand/MobiLipid
- name: RStudio 2023.09.1
  role: Interactive environment used to render the correction Rmd and inspect generated
    functions
- name: R 4.3.2
  role: Runtime for data filtering, evaluation, and visualization in the correction
    workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:a1be7ef00aecdc0b5e222bcb6f8d3bd841d48564d0ee6b8d17c6470a79e2b69a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FelinaHildebrand/MobiLipid
  tools:
  - MobiLipid
  - R
  - R 4.3.2
  - R Markdown
  - RStudio
  - RStudio 2023.09.1
  - ggbeeswarm
  - ggplot2
  - tcltk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# linear-regression-ccs-correction-fitting

## Summary

Fits lipid class- and adduct-specific CCS correction functions by resampling 3–6 U13C-labeled lipids per combination and regressing measured CCS values against DTCCSN2 library values, then applying the functions to correct all measured CCS values. Enables MobiLipid's internal CCS quality control without external calibrant measurements.

## When to use

Use when TIM-MS or DTIM-MS lipidomics CCS values show class-specific bias against a DTCCSN2 reference library and at least 3 U13C-labeled lipids per lipid class-adduct combination are available for regression.

## When NOT to use

- Fewer than 3 U13C-labeled lipids exist for a lipid class-adduct combination, since linear regression requires a minimum of 3 lipids per function (correction restricted to Cer, DG, HexCer, LPC, PA, PC, PE, PI, PS, TG)
- Input measured values are already corrected CCS values — this skill fits functions from uncorrected measured CCS vs library pairs and should not be reapplied to corrected output

## Inputs

- CSV with measured CCS values (headers: File, LipidClass, LipidSpecies, Adduct, Label, CCS)
- U13C_DT_CCS_library.csv of DTCCSN2 values for U13C-labeled yeast lipids

## Outputs

- Correction_functions table (100 linear functions per class-adduct with slopes/intercepts)
- Corrected_CCS_values table with per-function corrected CCS and post-correction bias
- CCS_bias_mean_by_function and CCS_bias_mean_all_functions summary tables
- Violin plots of CCS bias before and after correction by labeled-lipid count

## How to apply

For each lipid class-adduct combination (e.g. PC [M+H]+, TG [M+NH4]+) with ≥3 U13C-labeled lipids, resample 3 to 6 labeled lipids to build up to 100 distinct linear correction functions. Fit y = m*x + b using measured CCS values of U13C-labeled lipids as x and the DTCCSN2 library values as y, requiring a minimum of 3 lipids per regression. Inspect slopes for deviation from 1 — MobiLipid found functions 31, 93, and 98 (slopes 0.01, 0.46, 2.42) subpar for PC [M+H]+. Apply every valid function to all measured CCS values regardless of labeling status, then monitor post-correction bias between corrected CCS values and library values; functions using more labeled lipids (4–6 vs 3) yield more robust corrections approaching ~0% mean absolute bias.

## Related tools

- **MobiLipid** (R Markdown tool that resamples U13C-labeled lipids, fits the linear correction functions, applies them, and reports post-correction bias) — https://github.com/FelinaHildebrand/MobiLipid
- **R Markdown** (Execution format wrapping the CCS correction workflow (MobiLipid_CCS-bias-calculation_CCS-correction.Rmd)) — https://github.com/FelinaHildebrand/MobiLipid
- **RStudio 2023.09.1** (Interactive environment used to render the correction Rmd and inspect generated functions)
- **R 4.3.2** (Runtime for data filtering, evaluation, and visualization in the correction workflow)

## Examples

```
Rscript -e 'Rmd <- tcltk::tk_choose.files(); rmarkdown::render(Rmd, output_format=c("html_document","pdf_document"))' # with MobiLipid_CCS-bias-calculation_CCS-correction.Rmd
```

## Evaluation signals

- Regression slopes of generated functions should cluster near 1; outlier functions (e.g. slopes 0.01, 0.46, 2.42 for PC [M+H]+) should be flagged and excluded
- Mean absolute post-correction CCS bias across 10 lipid classes should approach the reported ~0% (from 0.78% positive-mode pre-correction bias)
- Mean absolute bias of 4–6-lipid functions must be lower than that of 3-lipid functions, confirming robustness scales with labeled-lipid count
- Each function must be built from at least 3, and at most 6, resampled U13C-labeled lipids per lipid class-adduct combination
- Bias between corrected CCS values and DTCCSN2 library values for U13C-labeled lipids must be recomputed to monitor correction outcome

## Limitations

- Only lipid classes with ≥3 labeled lipids per class-adduct combination can be corrected; adducts are restricted to [M+H], [M+NH4], [M+Na], [M-H], and [M+HCOO] per the class-adduct table
- Up to 100 resampled functions are generated, but some may be unstable (slopes far from 1, e.g. 0.01–2.42) and require inspection before acceptance
- CCS correction performance was demonstrated for TIM-MS measurements of an unlabeled yeast extract spiked with U13C-labeled standards; generalization to other IM technologies or matrices was not shown

## Evidence

- [readme] Linear fitting resamples 3–6 labeled lipids per class-adduct: "MobiLipid computes up to 100 distinct correction functions employing 3 to 6 lipids of a lipid class-adduct combination for linear regression."
- [readme] Regression x/y variables are measured vs library CCS: [evidence span withheld — anchor status: not_found]
- [readme] Minimum 3 lipids restricts correctable classes: "CCS correction functions are based on linear regression functions which require a minimum of 3 lipids within a lipid class-adduct combination which restricts the CCS correction to the following lipid"
- [readme] Correction applied regardless of labeling status: "After generation of all CCS correction functions, all measured CCS values are corrected, irrespective of their labeling status."
- [abstract] Mean absolute bias reduced to ~0% after correction: "the mean absolute CCSN2 biases of 10 lipid classes could be reduced to approximately 0%"
- [results] Outlier slopes 0.01, 0.46, 2.42 identify subpar functions: [evidence span withheld — anchor status: not_found]
