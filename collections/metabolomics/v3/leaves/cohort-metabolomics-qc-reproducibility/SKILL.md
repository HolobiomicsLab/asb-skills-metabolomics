---
name: cohort-metabolomics-qc-reproducibility
description: Use when a targeted LC–MS metabolomics/lipidomics workflow needs validation
  that an automated integrator's peak areas match vendor-software (Xcalibur) results
  across large cohort studies with QC replicates.
when_to_use_negative:
- Input is raw LC–MS data that has not yet been peak-integrated by TARDIS or a vendor
  tool — peak integration must happen first
- The comparison target is untargeted preprocessing (xcms, MZmine, OpenMS) rather
  than targeted peak-area agreement with vendor software
- No QC replicate structure exists in the data, making CV-based reproducibility comparison
  impossible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TARDIS
  role: Automated targeted peak integration of LC–MS raw data producing the log2 peak
    areas under evaluation
  repo: https://github.com/UGent-LIMET/TARDIS
- name: Xcalibur
  role: Vendor software used by an experienced analyst for manual peak integration,
    serving as the reference comparison
- name: R
  role: Environment for computing Spearman correlations, Gaussian kernel regression,
    and paired CV comparisons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:2b1c5b3496a70fb2832b1ce2a006edcdcaae81620d2dbf38fc8aba812ba3675f
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
yes_rate: 0.9333333333333333
n_claims: 15
---

# cohort-metabolomics-qc-reproducibility

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitatively verifies that automated peak areas from the TARDIS R package agree with expert manual integration in Thermo Xcalibur, by recomputing Spearman rho, regression R2/slope on log2 peak areas and comparing QC-replicate coefficients of variation across the FAME, ENVIRONAGE and FGFP cohorts.

## When to use

Use when a targeted LC–MS metabolomics/lipidomics workflow needs validation that an automated integrator's peak areas match vendor-software (Xcalibur) results across large cohort studies with QC replicates.

## When NOT to use

- Input is raw LC–MS data that has not yet been peak-integrated by TARDIS or a vendor tool — peak integration must happen first
- The comparison target is untargeted preprocessing (xcms, MZmine, OpenMS) rather than targeted peak-area agreement with vendor software
- No QC replicate structure exists in the data, making CV-based reproducibility comparison impossible

## Inputs

- per-target log2 peak-area tables from TARDIS (FAME, ENVIRONAGE, FGFP cohorts)
- per-target log2 peak-area tables from Xcalibur manual integration (same cohorts)
- QC replicate annotations per cohort
- R analysis scripts from UGent-LIMET/tardis_publication_code

## Outputs

- tardis_vs_xcalibur_statistics.csv with per-target Spearman rho, regression R2, slope, and per-cohort CV comparison p-values
- mean rho, mean R2 and slope summary statistics vs published benchmarks
- paired-test p-values for CV medians per cohort (ENVIRONAGE/FGFP, FAME)

## How to apply

Load the deposited per-target log2 peak-area tables produced by TARDIS and by Xcalibur from the UGent-LIMET/tardis_publication_code repository (data on Zenodo, doi 10.5281/zenodo.14548032), covering the FAME, ENVIRONAGE and FGFP cohorts. Compute per-target Spearman rank correlations between the two log2 peak-area vectors and report the mean rho (benchmark: 0.96). Fit a Gaussian (kernel) regression of TARDIS log2 peak areas on Xcalibur log2 peak areas to extract R2 (benchmark: 0.94) and slope (benchmark: 0.97). For reproducibility, compute per-target coefficients of variation across QC replicates within each cohort and compare TARDIS vs Xcalibur CV medians with a paired test, expecting p < 0.05 for ENVIRONAGE/FGFP and p = 0.06 for FAME. Compile all recomputed statistics into a single CSV (e.g. tardis_vs_xcalibur_statistics.csv) and check each value against the published benchmarks.

## Related tools

- **TARDIS** (Automated targeted peak integration of LC–MS raw data producing the log2 peak areas under evaluation) — https://github.com/UGent-LIMET/TARDIS
- **Xcalibur** (Vendor software used by an experienced analyst for manual peak integration, serving as the reference comparison)
- **R** (Environment for computing Spearman correlations, Gaussian kernel regression, and paired CV comparisons)

## Examples

```
BiocManager::install("pablovgd/TARDIS", build_vignettes = FALSE, dependencies=TRUE); library(TARDIS); runTardis()
```

## Evaluation signals

- Mean per-target Spearman rho between TARDIS and Xcalibur log2 peak areas ≈ 0.96
- Mean regression R2 of TARDIS on Xcalibur log2 peak areas ≈ 0.94 with slope ≈ 0.97
- Paired CV-comparison p-values: p < 0.05 for ENVIRONAGE/FGFP, p = 0.06 for FAME
- Output CSV contains one row per target with all four statistic types populated and no missing values
- Recomputed values match the benchmarks in tardis_vs_xcalibur_statistics.csv within numerical tolerance

## Limitations

- FAME cohort CV comparison did not reach significance (p = 0.06), so agreement is cohort-dependent
- The article does not name or version the exact Xcalibur build used, so exact vendor-version reproduction is not guaranteed
- Agreement is assessed on log2-transformed peak areas from three specific cohorts (FAME, ENVIRONAGE, FGFP) and two LC–MS instruments; generalization to other assays is untested

## Evidence

- [other] Correlation and R2 between log2 peak areas reported by TARDIS and Xcalibur were high (mean ρ = 0.96, mean R2 = 0.94): [evidence span withheld — anchor status: not_found]
