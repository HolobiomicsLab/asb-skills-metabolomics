---
name: gaussian-kernel-regression-agreement-fitting
description: Use when you have matched per-target log2 peak-area tables from two integration
  methods (e.g., an open-source R pipeline vs. vendor software such as Xcalibur) and
  need to quantify whether the automated tool reproduces expert manual integration.
when_to_use_negative:
- Input is untargeted feature-table output (e.g., from xcms, MZmine, or OpenMS) rather
  than matched targeted peak areas from two integration methods
- There is no vendor/expert reference integration to compare against, so agreement
  cannot be assessed
- Peak areas are on the raw (non-log2) scale and no matched QC replicates exist, making
  Spearman/kernel-regression and CV comparison inappropriate as implemented
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Statistical environment used to compute Spearman correlations, Gaussian kernel
    regression fits, CVs, and paired tests on log2 peak-area tables
  repo: https://github.com/UGent-LIMET/TARDIS
- name: TARDIS
  role: Open-source R package that produces the automated per-target log2 peak areas
    being validated against vendor software
  repo: https://github.com/UGent-LIMET/TARDIS
- name: Xcalibur
  role: Thermo vendor software used by an experienced analyst to produce the reference
    log2 peak areas for the agreement comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:e2d6cf860f9732264a81703f17352eb696585abe60c631a1c9e9c91079c590be
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gaussian-kernel-regression-agreement-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitative agreement analysis between automated (TARDIS) and vendor (Xcalibur) log2 peak areas, using per-target Spearman correlation, Gaussian kernel regression (R2 and slope), and paired comparison of QC-replicate coefficients of variation across cohorts.

## When to use

Use when you have matched per-target log2 peak-area tables from two integration methods (e.g., an open-source R pipeline vs. vendor software such as Xcalibur) and need to quantify whether the automated tool reproduces expert manual integration.

## When NOT to use

- Input is untargeted feature-table output (e.g., from xcms, MZmine, or OpenMS) rather than matched targeted peak areas from two integration methods
- There is no vendor/expert reference integration to compare against, so agreement cannot be assessed
- Peak areas are on the raw (non-log2) scale and no matched QC replicates exist, making Spearman/kernel-regression and CV comparison inappropriate as implemented

## Inputs

- Per-target log2 peak-area tables for TARDIS (automated integration) from the FAME, ENVIRONAGE, and FGFP cohorts
- Per-target log2 peak-area tables for Xcalibur (expert analyst vendor integration) for the same cohorts
- QC replicate peak areas per cohort for coefficient-of-variation computation

## Outputs

- tardis_vs_xcalibur_statistics.csv with per-target Spearman rho, Gaussian kernel regression R2 and slope, and QC CV medians with paired-test p-values

## How to apply

Load the deposited per-target log2 peak-area tables for TARDIS and Xcalibur from the UGent-LIMET/tardis_publication_code repository (Zenodo input data), covering the FAME, ENVIRONAGE, and FGFP cohorts. First compute the per-target Spearman rank correlation between TARDIS and Xcalibur log2 peak areas and report the mean rho (benchmark: mean rho = 0.96). Next fit a Gaussian (kernel) regression of TARDIS log2 peak areas on Xcalibur log2 peak areas and extract the resulting R2 and slope (benchmarks: mean R2 = 0.94, slope = 0.97), which quantifies whether the automated boundaries yield areas proportional to the vendor's. Then compute per-target coefficients of variation across QC replicates within each cohort and compare TARDIS vs Xcalibur medians with a paired statistical test, recording p-values separately for ENVIRONAGE/FGFP (expected p < 0.05) and FAME (expected p = 0.06). Compile all recomputed statistics into tardis_vs_xcalibur_statistics.csv and check each value against the reported benchmarks.

## Related tools

- **R** (Statistical environment used to compute Spearman correlations, Gaussian kernel regression fits, CVs, and paired tests on log2 peak-area tables) — https://github.com/UGent-LIMET/TARDIS
- **TARDIS** (Open-source R package that produces the automated per-target log2 peak areas being validated against vendor software) — https://github.com/UGent-LIMET/TARDIS
- **Xcalibur** (Thermo vendor software used by an experienced analyst to produce the reference log2 peak areas for the agreement comparison)

## Examples

```
ksmooth(xcalibur_log2_areas, tardis_log2_areas, kernel = "normal", bandwidth = bw); summary(fit)$r.squared
```

## Evaluation signals

- Mean per-target Spearman rho between TARDIS and Xcalibur log2 peak areas approximates 0.96
- Mean Gaussian kernel regression R2 approximates 0.94 with regression slope approximating 0.97
- Paired CV comparison yields p < 0.05 for ENVIRONAGE/FGFP and p = 0.06 for FAME
- Output CSV (tardis_vs_xcalibur_statistics.csv) contains all recomputed statistics and each value matches the reported benchmarks
- Statistics are computed per target and reported as cohort-specific results for FAME, ENVIRONAGE, and FGFP

## Limitations

- The comparison vendor software is only described generically as 'state-of-the-art vendor software' in parts of the article and is never versioned, complicating exact reproduction
- No version number for the TARDIS R package is stated; the README requires R >= 4.4.0 and BiocManager 3.20 for installation
- The FAME cohort CV comparison (p = 0.06) does not reach significance, so 'CV parity' is cohort-dependent rather than universal

## Evidence

- [abstract] The robustness of TARDIS' peak integration was demonstrated through a quantitative comparison to state-of-the-art vendor software.: [evidence span withheld — anchor status: not_found]
- [readme] The publication-code repository provides the scripts to reproduce the statistics.: [evidence span withheld — anchor status: external_doc]
