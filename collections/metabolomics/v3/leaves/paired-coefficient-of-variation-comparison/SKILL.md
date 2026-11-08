---
name: paired-coefficient-of-variation-comparison
description: Use when two peak-integration approaches must be quantitatively compared
  on log2 peak areas per target, and replicate QC data are available to compute per-target
  CVs for a paired significance test.
when_to_use_negative:
- Data are untargeted peak tables from xcms, MZmine, or OpenMS rather than targeted
  integrated peak areas from TARDIS and a vendor tool
- No QC replicate structure exists, so per-target coefficients of variation cannot
  be computed for a paired comparison
- Peak areas are on the linear scale and the log2 transformation used by the article
  has not been applied, making results incomparable to the published benchmarks
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: TARDIS
  role: open-source R package producing the automated log2 peak areas under evaluation
  repo: https://github.com/UGent-LIMET/TARDIS
- name: Xcalibur
  role: vendor software whose manually integrated log2 peak areas serve as the analyst
    reference
- name: R
  role: language used to compute Spearman correlations, kernel regression, CVs, and
    the paired statistical test
  repo: UGent-LIMET/tardis_publication_code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:ea05111a85d5ddde169c908e639ee483c2cf8eff5753f6d1eaa0b0e6cfaa637a
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# paired-coefficient-of-variation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks automated peak integration (TARDIS) against vendor software (Xcalibur) by comparing per-target coefficients of variation across QC replicates with a paired statistical test, alongside Spearman rho, regression R2, and slope of log2 peak areas.

## When to use

Use when two peak-integration approaches must be quantitatively compared on log2 peak areas per target, and replicate QC data are available to compute per-target CVs for a paired significance test.

## When NOT to use

- Data are untargeted peak tables from xcms, MZmine, or OpenMS rather than targeted integrated peak areas from TARDIS and a vendor tool
- No QC replicate structure exists, so per-target coefficients of variation cannot be computed for a paired comparison
- Peak areas are on the linear scale and the log2 transformation used by the article has not been applied, making results incomparable to the published benchmarks

## Inputs

- per-target log2 peak-area tables from TARDIS (FAME, ENVIRONAGE, FGFP cohorts)
- per-target log2 peak-area tables from Xcalibur (FAME, ENVIRONAGE, FGFP cohorts)
- QC replicate annotations per cohort to group runs for CV computation

## Outputs

- tardis_vs_xcalibur_statistics.csv with per-target Spearman rho, mean rho, mean R2, slope, and per-cohort paired-test p-values
- per-target coefficients of variation across QC replicates for TARDIS and Xcalibur

## How to apply

Load the deposited per-target log2 peak-area tables for TARDIS and Xcalibur from UGent-LIMET/tardis_publication_code (data on Zenodo, doi 10.5281/zenodo.14548032), covering the FAME, ENVIRONAGE, and FGFP cohorts. Compute per-target Spearman rank correlation between TARDIS and Xcalibur log2 peak areas, then fit a Gaussian (kernel) regression of TARDIS on Xcalibur log2 peak areas to extract R2 and slope. Next compute per-target coefficients of variation across QC replicates within each cohort and compare TARDIS vs Xcalibur CV medians with a paired statistical test, recording p-values separately for ENVIRONAGE/FGFP and FAME. Compile all recomputed statistics into tardis_vs_xcalibur_statistics.csv and verify each value against the published benchmarks.

## Related tools

- **TARDIS** (open-source R package producing the automated log2 peak areas under evaluation) — https://github.com/UGent-LIMET/TARDIS
- **Xcalibur** (vendor software whose manually integrated log2 peak areas serve as the analyst reference)
- **R** (language used to compute Spearman correlations, kernel regression, CVs, and the paired statistical test) — UGent-LIMET/tardis_publication_code

## Examples

```
wilcox.test(cv_tardis, cv_xcalibur, paired = TRUE)  # per-target CVs across QC replicates, TARDIS vs Xcalibur
```

## Evaluation signals

- mean Spearman rho between TARDIS and Xcalibur log2 peak areas equals ~0.96
- mean R2 from the Gaussian (kernel) regression equals ~0.94 with slope ~0.97
- paired test comparing TARDIS vs Xcalibur CV medians gives p < 0.05 for ENVIRONAGE/FGFP and p = 0.06 for FAME
- tardis_vs_xcalibur_statistics.csv contains all recomputed statistics and each matches the reported benchmarks

## Limitations

- In the discussion section the comparison vendor software is only described generically as 'state-of-the-art vendor software' and never named or versioned, so replication depends on the workflow card identifying Xcalibur
- The FAME cohort comparison was not significant (p = 0.06), so CV agreement should not be claimed for that cohort
- No version number for the TARDIS R package is stated in the article; R >= 4.4.0 and BiocManager 3.20 are required per the README

## Evidence

- [abstract] The robustness of TARDIS' peak integration was demonstrated through a quantitative comparison to state-of-the-art vendor software.: [evidence span withheld — anchor status: not_found]
- [readme] This repository contains all R and python scripts to reproduce the data-analysis and figures presented in Vangeenderhuysen et al. 2025.: [evidence span withheld — anchor status: external_doc]
