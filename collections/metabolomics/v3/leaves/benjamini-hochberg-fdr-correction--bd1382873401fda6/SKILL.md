---
name: benjamini-hochberg-fdr-correction
description: Use when a MOFA factor-interpretation step yields many nominal p-values
  from Pearson (numeric clinical features) and Wilcoxon (binary clinical features)
  tests that must be FDR-corrected before ranking discriminant factors.
when_to_use_negative:
- No multiple testing is involved — a single factor-feature hypothesis test needs
  no BH correction
- The goal is single-omics differential analysis (e.g. metabolomics peak differences
  use Wilcoxon with FDR in a separate BiomiX module) rather than MOFA factor interpretation
- MOFA factors are already annotated by the PubMed bibliography-mining or pathway
  (EnrichR/GSEA) interpretation routes and no clinical correlation table is requested
edam_operation: http://edamontology.org/operation_0391
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: BiomiX
  role: 'Implements the MOFA factor clinical-correlation module: Pearson correlation
    for numeric clinical data, Wilcoxon test for binary data, and Benjamini–Hochberg
    correction of nominal p-values'
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the factor values matrix that is correlated against the clinical
    metadata table
- name: R
  role: Language/environment in which the BiomiX correlation analysis and p-value
    correction are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:337ea2d19a731c7bf349e0612b7882965db2bf1586fa9364e58d31382ae53f9f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - MOFA (Multi-Omics Factor Analysis)
  - R
  edam_operation: http://edamontology.org/operation_0391
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# benjamini-hochberg-fdr-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Benjamini–Hochberg false discovery rate correction to nominal p-values produced when BiomiX correlates MOFA factors with clinical metadata (Pearson for numeric features, Wilcoxon for binary features), so discriminant factors can be ranked by clinical association without inflating type-I error across many factor-feature tests.

## When to use

Use when a MOFA factor-interpretation step yields many nominal p-values from Pearson (numeric clinical features) and Wilcoxon (binary clinical features) tests that must be FDR-corrected before ranking discriminant factors.

## When NOT to use

- No multiple testing is involved — a single factor-feature hypothesis test needs no BH correction
- The goal is single-omics differential analysis (e.g. metabolomics peak differences use Wilcoxon with FDR in a separate BiomiX module) rather than MOFA factor interpretation
- MOFA factors are already annotated by the PubMed bibliography-mining or pathway (EnrichR/GSEA) interpretation routes and no clinical correlation table is requested

## Inputs

- MOFA factor values matrix (factor scores per sample)
- Clinical metadata table with numeric and binary clinical features
- Nominal p-values from Pearson correlation of factors vs numeric clinical features
- Nominal p-values from Wilcoxon rank-sum test of factors vs binary clinical features

## Outputs

- mofa_factor_clinical_correlations.csv — per-factor significance table with test type, statistic, raw p-value, and BH-adjusted p-value for every factor-feature combination

## How to apply

Load the MOFA factor values matrix and the clinical metadata table in the BiomiX R environment. For each MOFA factor, compute Pearson correlation coefficients and p-values against every numeric clinical feature, and test every binary clinical feature with the Wilcoxon rank-sum test on the positive/negative sample groups. Pool all raw p-values from both the Pearson and Wilcoxon analyses into a single vector and apply Benjamini–Hochberg correction so that testing multiple factor-feature pairs simultaneously does not inflate false discovery. Assemble the per-factor significance table listing each factor-feature pair with its test type, statistic, raw p-value, and BH-adjusted p-value, and write it to mofa_factor_clinical_correlations.csv.

## Related tools

- **BiomiX** (Implements the MOFA factor clinical-correlation module: Pearson correlation for numeric clinical data, Wilcoxon test for binary data, and Benjamini–Hochberg correction of nominal p-values) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Produces the factor values matrix that is correlated against the clinical metadata table)
- **R** (Language/environment in which the BiomiX correlation analysis and p-value correction are executed)

## Examples

```
Rscript -e 'p <- read.csv("raw_pvalues.csv")$pval; adj <- p.adjust(p, method="BH"); write.csv(adj, "mofa_factor_clinical_correlations.csv")'
```

## Evaluation signals

- mofa_factor_clinical_correlations.csv exists and is a valid CSV
- The table contains an entry for every factor-feature combination tested (Pearson pairs for numeric features, Wilcoxon pairs for binary features)
- A BH-adjusted p-value column is present alongside each raw p-value, and adjusted values are >= raw values and bounded by [0, 1]
- Each row records the correct test type (Pearson for numeric clinical features, Wilcoxon for binary) and its statistic
- Discriminant MOFA factors can be ranked by their BH-adjusted association with clinical data

## Limitations

- BiomiX does not automatically validate datasets; users are strongly recommended to carefully review the clinical dataset before running the correlation and correction step
- The correlation/bibliography analysis bug fixed in v2.4 shows the module must correctly recognize 'undefined omics' contributors — older versions may miss such factor contributors
- The tool is described as 'a first attempt', not a definitive solution, so results of the factor-clinical correlation table should be interpreted as exploratory

## Evidence

- [full_text] BiomiX correlates numerical clinical data with each MOFA factor using Pearson correlation, tests binary clinical data with the Wilcoxon test on positive/negative groups, and corrects nominal p-values using the Benjamini–Hochberg method.: [evidence span withheld — anchor status: not_found]
- [abstract] Correlation of discriminant MOFA factors with clinical data and exploration of top contributing pathways: "discriminant MOFA factors are correlated with clinical data, and the top contributing pathways are explored"
- [readme] MOFA factor interpretation bug fix in BiomiX v2.4 README: "Bug fixed: the bibliography and correlation analysis are now properly recognizing the undefined omics contributors to correlate and interpret them."
