---
name: spearman-correlation-analysis-of-peak-areas
description: Use when two peak-integration approaches have produced matched per-target
  peak-area tables (e.g., log2 AUC per target per run) and you must demonstrate their
  quantitative agreement via rank correlation, regression, and precision (CV) benchmarking.
when_to_use_negative:
- Input is untargeted feature data (e.g., xcms/MZmine/OpenMS output) rather than matched
  per-target peak areas from two integration methods
- Only one integration method is available, so no TARDIS-vs-vendor agreement comparison
  is possible
- Data are not log2-transformed peak areas or contain unmatched targets, making rank
  correlation and CV comparison invalid
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Language used to run the Spearman correlation, Gaussian kernel regression,
    and paired CV comparison statistics
  repo: https://github.com/UGent-LIMET/TARDIS
- name: TARDIS
  role: Source of the automated peak-integration log2 peak areas being validated
  repo: https://github.com/UGent-LIMET/TARDIS
- name: Xcalibur
  role: Vendor software providing the expert-analyst reference peak areas for comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00567
    title: Automated Integration and Quality Assessment of Chromatographic Peaks in
      LC–MS-Based Metabolomics and Lipidomics Using TARDIS
  source_content_hash: sha256:99d758ba2bea4c7dfb4caa56a35044bf62c054e38c6718abc79260b409727cd4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spearman-correlation-analysis-of-peak-areas

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitatively validates automated peak integration (TARDIS) against expert vendor-software integration (Xcalibur) by computing per-target Spearman rank correlations, Gaussian-kernel regression R2/slope, and QC-replicate CV comparisons on log2 peak areas across the FAME, ENVIRONAGE, and FGFP cohorts.

## When to use

Use when two peak-integration approaches have produced matched per-target peak-area tables (e.g., log2 AUC per target per run) and you must demonstrate their quantitative agreement via rank correlation, regression, and precision (CV) benchmarking.

## When NOT to use

- Input is untargeted feature data (e.g., xcms/MZmine/OpenMS output) rather than matched per-target peak areas from two integration methods
- Only one integration method is available, so no TARDIS-vs-vendor agreement comparison is possible
- Data are not log2-transformed peak areas or contain unmatched targets, making rank correlation and CV comparison invalid

## Inputs

- Per-target log2 peak-area table produced by TARDIS (AUC of each target in each run; results object is a list containing a data.frame)
- Per-target log2 peak-area table produced by Xcalibur vendor integration
- QC replicate annotations per cohort (FAME, ENVIRONAGE, FGFP)

## Outputs

- tardis_vs_xcalibur_statistics.csv with per-target Spearman rho, mean rho, Gaussian regression R2 and slope, and paired-test CV comparison p-values per cohort

## How to apply

Load the deposited per-target log2 peak-area tables for TARDIS and Xcalibur (available via the UGent-LIMET/tardis_publication_code repository, with input data on Zenodo doi 10.5281/zenodo.14548032) covering the FAME, ENVIRONAGE, and FGFP cohorts. Compute the per-target Spearman rank correlation between TARDIS and Xcalibur log2 peak areas and report the mean rho; benchmark against mean rho = 0.96. Fit a Gaussian (kernel) regression of TARDIS log2 peak areas on Xcalibur log2 peak areas and extract R2 and slope, expected near R2 = 0.94 and slope = 0.97. Compute per-target coefficients of variation across QC replicates within each cohort and compare TARDIS vs Xcalibur medians with a paired test, recording p-values for ENVIRONAGE/FGFP (expected p < 0.05) and FAME (expected p = 0.06). Compile all recomputed statistics into tardis_vs_xcalibur_statistics.csv and check each value against these reported benchmarks.

## Related tools

- **R** (Language used to run the Spearman correlation, Gaussian kernel regression, and paired CV comparison statistics) — https://github.com/UGent-LIMET/TARDIS
- **TARDIS** (Source of the automated peak-integration log2 peak areas being validated) — https://github.com/UGent-LIMET/TARDIS
- **Xcalibur** (Vendor software providing the expert-analyst reference peak areas for comparison)

## Examples

```
cor.test(tardis_log2_area, xcalibur_log2_area, method = "spearman", exact = FALSE)
```

## Evaluation signals

- Mean per-target Spearman rho equals or closely matches the reported benchmark of 0.96
- Gaussian kernel regression yields R2 near 0.94 and slope near 0.97
- Paired CV comparison gives p < 0.05 for ENVIRONAGE/FGFP and p = 0.06 for FAME
- All values recomputed into tardis_vs_xcalibur_statistics.csv match the reported benchmarks within expected tolerance
- Output covers all three cohorts (FAME, ENVIRONAGE, FGFP) from the 1786-run benchmark dataset

## Limitations

- Comparison vendor software is described generically as 'state-of-the-art vendor software' in parts of the article without name or version; version tracking depends on the task metadata
- The FAME cohort CV comparison was not significant (p = 0.06), so agreement in precision is cohort-dependent
- Results depend on deposited data (Zenodo doi 10.5281/zenodo.14548032); without it the statistics cannot be reproduced

## Evidence

- [results] Correlation and R2 between log2 peak areas reported by TARDIS and Xcalibur were high: [evidence span withheld — anchor status: not_found]
- [readme] publication code repository README: [evidence span withheld — anchor status: external_doc]
