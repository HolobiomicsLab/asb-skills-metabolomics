---
name: per-ionization-mode-statistical-aggregation
description: Use when a CCS bias table (per lipid species-adduct, in %) has been computed
  against a reference CCS library and you need a single quality-control metric per
  ionization mode (positive vs negative).
when_to_use_negative:
- When the measured data are not from an IM-MS platform with U13C-labeled internal
  standards matched to the DTCCSN2 library — the aggregation is only meaningful for
  bias values computed against this reference
- When lipid class-adduct combinations fall outside the supported table (e.g. unsupported
  adducts per lipid class) — per-lipid biases would not be valid inputs to the aggregation
- When the goal is CCS correction rather than bias assessment — use the CCS-correction
  workflow (linear regression functions, ≥3 lipids per class-adduct) instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MobiLipid
  role: R Markdown tool that performs the per-lipid CCS bias calculation and produces
    the CCS_bias_no_correction table aggregated per ionization mode
  repo: github.com/FelinaHildebrand/MobiLipid
- name: R
  role: Runtime environment (R 4.3.2) in which the bias aggregation and per-mode mean
    summaries are computed
  repo: https://cran.r-project.org/
- name: RStudio
  role: Environment (RStudio 2023.09.1) used to run the R Markdown and inspect the
    aggregated bias outputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c01253
    title: 'MobiLipid: A Tool for Enhancing CCS Quality Control of Ion Mobility–Mass
      Spectrometry Lipidomics by Internal Standardization'
  source_content_hash: sha256:08e2bc37b3b1c0c658f9a42b066bcd1a4582313bfa79f00f1951c04c329857b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MobiLipid
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Per-ionization-mode statistical aggregation

## Summary

Aggregates per-lipid absolute CCSN2 bias values (measured TIMCCSN2 vs the U13C DTCCSN2 library reference) into mean absolute bias summarized separately for positive and negative ionization mode, yielding the benchmark metrics 0.78% (positive) and 0.33% (negative).

## When to use

Use when a CCS bias table (per lipid species-adduct, in %) has been computed against a reference CCS library and you need a single quality-control metric per ionization mode (positive vs negative).

## When NOT to use

- When the measured data are not from an IM-MS platform with U13C-labeled internal standards matched to the DTCCSN2 library — the aggregation is only meaningful for bias values computed against this reference
- When lipid class-adduct combinations fall outside the supported table (e.g. unsupported adducts per lipid class) — per-lipid biases would not be valid inputs to the aggregation
- When the goal is CCS correction rather than bias assessment — use the CCS-correction workflow (linear regression functions, ≥3 lipids per class-adduct) instead

## Inputs

- Per-lipid CCS bias table (CCS_bias_no_correction) with columns File, LipidClass, LipidSpecies, Adduct, Label ('heavy'), CCS, and computed bias (%)
- U13C_DT_CCS_library.csv (DTCCSN2 reference library for U13C-labeled yeast lipids, 377 values)
- Lipid class-adduct compatibility mapping (e.g. PC: [M+H], [M+Na], [M+HCOO])

## Outputs

- Mean absolute CCSN2 bias per ionization mode (positive ≈ 0.78%, negative ≈ 0.33% for the benchmark dataset)
- CCS_bias_no_correction summary table (.csv) and rendered .html/.pdf report from the R Markdown

## How to apply

After MobiLipid ('MobiLipid_CCS-bias-calculation.Rmd') matches each 'heavy' (U13C-labeled) lipid species-adduct to its DTCCSN2 library entry and computes per-lipid CCS bias (%) with the library value as reference, restrict to entries labeled 'heavy' and to lipid class-adduct combinations valid per the compatibility table (e.g. PC with [M+H], [M+Na], [M+HCOO]). Aggregate the absolute per-lipid biases into the CCS_bias_no_correction table, then compute the mean absolute bias summarized by ionization mode so that positive- and negative-mode performance are reported independently rather than pooled. Compare the resulting per-mode means against the published benchmark (≈0.78% positive, ≈0.33% negative) to judge whether the TIM-MS data are within expected agreement with the DTIM-MS-derived library; the same aggregation is reused post-correction to confirm bias reduction toward 0%.

## Related tools

- **MobiLipid** (R Markdown tool that performs the per-lipid CCS bias calculation and produces the CCS_bias_no_correction table aggregated per ionization mode) — github.com/FelinaHildebrand/MobiLipid
- **R** (Runtime environment (R 4.3.2) in which the bias aggregation and per-mode mean summaries are computed) — https://cran.r-project.org/
- **RStudio** (Environment (RStudio 2023.09.1) used to run the R Markdown and inspect the aggregated bias outputs)

## Examples

```
rmarkdown::render(Rmd, output_format = c("html_document", "pdf_document"), output_file = c(paste0(Rmd_name,"_",filename,".html"), paste0(Rmd_name,"_",filename,".pdf")))
```

## Evaluation signals

- Computed mean absolute CCSN2 bias per ionization mode should reproduce approximately 0.78% (positive) and 0.33% (negative) on the reference TIM-MS dataset
- All aggregated rows must carry Label == 'heavy'; light (natural) lipids must be excluded from the bias aggregation
- Every matched species-adduct must be a valid lipid class-adduct combination from the compatibility table
- The CCS_bias_no_correction output table must exist as a .csv and be fully accessible in the .html report (PDF shows only the first 10 rows per table)
- After applying CCS correction, the same aggregation over 10 lipid classes should show mean absolute biases reduced to approximately 0%

## Limitations

- MobiLipid has no specified software version and no repository changelog
- Only lipid class-adduct combinations listed in the compatibility table are valid; unsupported combinations silently fall outside the analysis
- The benchmark values (0.78%/0.33%) are specific to TIM-MS vs DTIM-MS comparison with DTCCSN2 as reference; other platform pairs will differ

## Evidence

- [abstract] mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively: "mean absolute biases of 0.78% and 0.33% in positive and negative ionization mode, respectively"
- [readme] MobiLipid bias calculation: "it is possible to utilize the <sup>DT</sup>CCS<sub>N2</sub> library for U<sup>13</sup>C labeled lipids (provided together with the code and publication) to first calculate CCS bias between measured"
- [intro] library quality thresholds: "(4) a CCS relative standard deviation <1%"
- [readme] bias table output: "*CCS_bias_no_correction*: Table with CCS bias (%) before correction"
