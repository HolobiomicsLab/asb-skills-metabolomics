---
name: lc-ms-vendor-software-benchmarking
description: Use when peak areas from an automated integrator must be validated against
  an experienced analyst's vendor-software (e.g., Xcalibur) peak areas using correlation,
  regression, and reproducibility metrics.
when_to_use_negative:
- Input is raw LC-MS vendor files that have not yet been peak-integrated by both TARDIS
  and the vendor software — run the integration workflows first
- The comparison is against untargeted preprocessing tools (xcms, MZmine, OpenMS)
  rather than targeted vendor software; this skill benchmarks TARDIS vs vendor tools
  like Xcalibur specifically
- Only one integrator's peak areas are available, so no paired agreement statistics
  can be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TARDIS
  role: Open-source R package whose log2 peak areas are benchmarked against vendor
    software output
  repo: https://github.com/UGent-LIMET/TARDIS
- name: Xcalibur
  role: Thermo vendor software used by an experienced analyst as the reference integration
    for peak-area comparison
- name: R
  role: Language used to load peak-area tables, compute Spearman correlations, kernel
    regression R2/slope, and paired CV tests
  repo: https://github.com/UGent-LIMET/TARDIS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:2e435c6f03d53a380acdb549457b53869c90980eaaa196a4517b6d2952c2c5d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/UGent-LIMET/TARDIS
  repo_url: https://github.com/UGent-LIMET/TARDIS
  tools:
  - R
  - TARDIS
  - Xcalibur
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lc-ms-vendor-software-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitatively benchmark an open-source targeted LC-MS peak integration pipeline (TARDIS) against vendor software (Xcalibur) by recomputing agreement statistics — Spearman rho, regression R2, slope, and QC replicate CV comparisons — on log2 peak areas across cohorts.

## When to use

Use when peak areas from an automated integrator must be validated against an experienced analyst's vendor-software (e.g., Xcalibur) peak areas using correlation, regression, and reproducibility metrics.

## When NOT to use

- Input is raw LC-MS vendor files that have not yet been peak-integrated by both TARDIS and the vendor software — run the integration workflows first
- The comparison is against untargeted preprocessing tools (xcms, MZmine, OpenMS) rather than targeted vendor software; this skill benchmarks TARDIS vs vendor tools like Xcalibur specifically
- Only one integrator's peak areas are available, so no paired agreement statistics can be computed

## Inputs

- Per-target log2 peak-area tables for TARDIS and Xcalibur from UGent-LIMET/tardis_publication_code (input data on Zenodo, doi 10.5281/zenodo.14548032)
- Peak areas spanning the FAME, ENVIRONAGE, and FGFP cohort studies
- QC replicate annotations within each cohort for CV computation

## Outputs

- tardis_vs_xcalibur_statistics.csv with per-target Spearman rho, regression R2, slope, and CV comparison p-values
- Mean agreement statistics (rho, R2, slope) comparable to reported benchmarks (rho = 0.96, R2 = 0.94, slope = 0.97)

## How to apply

Load the deposited per-target log2 peak-area tables for both the automated integrator (TARDIS) and vendor software (Xcalibur) from the UGent-LIMET/tardis_publication_code repository (data on Zenodo, doi 10.5281/zenodo.14548032), covering the FAME, ENVIRONAGE, and FGFP cohorts. Compute per-target Spearman rank correlations between the two sets of log2 peak areas and report the mean rho (expected benchmark: mean rho = 0.96). Fit a Gaussian (kernel) regression of TARDIS log2 peak areas on Xcalibur log2 peak areas and extract R2 (expected mean 0.94) and slope (expected 0.97). Compute per-target coefficients of variation across QC replicates within each cohort, compare TARDIS vs Xcalibur medians with a paired statistical test, and record p-values (ENVIRONAGE/FGFP p < 0.05; FAME p = 0.06). Compile all recomputed statistics into a results table (tardis_vs_xcalibur_statistics.csv) and check each value against the reported benchmarks.

## Related tools

- **TARDIS** (Open-source R package whose log2 peak areas are benchmarked against vendor software output) — https://github.com/UGent-LIMET/TARDIS
- **Xcalibur** (Thermo vendor software used by an experienced analyst as the reference integration for peak-area comparison)
- **R** (Language used to load peak-area tables, compute Spearman correlations, kernel regression R2/slope, and paired CV tests) — https://github.com/UGent-LIMET/TARDIS

## Examples

```
Rscript scripts/recompute_agreement.R --input tardis_vs_xcalibur_log2_areas.csv --output tardis_vs_xcalibur_statistics.csv
```

## Evaluation signals

- Mean Spearman rho between TARDIS and Xcalibur log2 peak areas is ~0.96
- Mean regression R2 of TARDIS on Xcalibur log2 peak areas is ~0.94 with slope ~0.97
- Paired CV comparison yields p < 0.05 for ENVIRONAGE/FGFP cohorts and p = 0.06 for FAME
- Recomputed values match the reported benchmarks when checked against tardis_vs_xcalibur_statistics.csv
- Statistics are computed per-target across all three cohorts (FAME, ENVIRONAGE, FGFP) before aggregation

## Limitations

- Vendor software comparison is only described generically as 'state-of-the-art vendor software' in parts of the article; software name and version should be recorded explicitly when reproducing
- Agreement is assessed on log2 peak areas across specific cohorts and instruments; generalization to other biofluids or LC-MS instruments is not established by these benchmarks alone
- FAME cohort CV comparison did not reach conventional significance (p = 0.06), so reproducibility conclusions are cohort-dependent

## Evidence

- [results] Correlation and R2 between log2 peak areas reported by TARDIS and Xcalibur were high (mean rho = 0.96, mean R2 = 0.94) in comparison to similar studies, indicating accurate peak boundary determination.: [evidence span withheld — anchor status: not_found]
- [readme] Reproduction scripts repository: [evidence span withheld — anchor status: external_doc]
