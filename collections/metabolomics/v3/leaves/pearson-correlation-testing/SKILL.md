---
name: pearson-correlation-testing
description: Use when interpreting discriminant MOFA (or DIABLO) factors by testing
  their association with clinical metadata — numeric features via Pearson correlation,
  binary features via Wilcoxon — inside the BiomiX R pipeline.
when_to_use_negative:
- The goal is biological process annotation of factor contributors — BiomiX deliberately
  complements correlation with EnrichR/GSEA pathway analysis and PubMed bibliography
  mining, so use those modules instead of correlation alone
- Clinical features are neither numeric nor binary (e.g., multi-level categorical)
  — the module specifies Pearson for numeric and Wilcoxon for binary data only
- No MOFA (or DIABLO) factor matrix is available because integration was performed
  by another method without factor values (e.g., raw SNF/NEMO clustering outputs)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: BiomiX
  role: R-based tool whose MOFA factor interpretation module runs the Pearson correlation,
    Wilcoxon test, and Benjamini–Hochberg correction against clinical data
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the factor values matrix that is correlated with clinical features
- name: R
  role: Environment in which the correlation/Wilcoxon testing and BH correction are
    executed (cor.test, wilcox.test, p.adjust)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:68d5bd5574ebe18d1c34978db747a9ff8427db815d281b0831e09f3ca6267389
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pearson-correlation-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements BiomiX's MOFA factor–clinical data correlation module: Pearson correlation of each MOFA factor against numeric clinical features and Wilcoxon rank-sum testing of binary clinical features, with Benjamini–Hochberg FDR correction across all pooled p-values to interpret discriminant factors.

## When to use

Use when interpreting discriminant MOFA (or DIABLO) factors by testing their association with clinical metadata — numeric features via Pearson correlation, binary features via Wilcoxon — inside the BiomiX R pipeline.

## When NOT to use

- The goal is biological process annotation of factor contributors — BiomiX deliberately complements correlation with EnrichR/GSEA pathway analysis and PubMed bibliography mining, so use those modules instead of correlation alone
- Clinical features are neither numeric nor binary (e.g., multi-level categorical) — the module specifies Pearson for numeric and Wilcoxon for binary data only
- No MOFA (or DIABLO) factor matrix is available because integration was performed by another method without factor values (e.g., raw SNF/NEMO clustering outputs)

## Inputs

- MOFA factor values matrix (factors × samples) produced by the BiomiX MOFA integration step
- Clinical metadata table with numeric features (e.g., age, lab values) and binary features (e.g., cohort/disease status)

## Outputs

- mofa_factor_clinical_correlations.csv: per-factor significance table with test type, statistic, raw p-value, and BH-adjusted p-value for every factor-feature combination
- Ranked list of discriminant MOFA factors by clinical association

## How to apply

Load the MOFA factor values matrix and the clinical metadata table into the BiomiX R environment. For every MOFA factor, compute Pearson correlation coefficients and raw p-values against each numeric clinical feature in the metadata, and test each binary clinical feature with the Wilcoxon rank-sum test comparing the positive/negative factor groups. Pool the raw p-values from both the Pearson and Wilcoxon analyses and apply Benjamini–Hochberg false discovery rate correction, so that factor-feature significance is judged jointly rather than per test. Assemble a per-factor significance table listing each factor-feature pair with its test type, statistic, raw p-value, and BH-adjusted p-value, and write it to mofa_factor_clinical_correlations.csv. This correlation module is one of BiomiX's three factor-interpretation approaches (alongside PubMed bibliography mining and pathway analysis), so rank discriminant factors by BH-adjusted clinical association rather than relying on correlation alone.

## Related tools

- **BiomiX** (R-based tool whose MOFA factor interpretation module runs the Pearson correlation, Wilcoxon test, and Benjamini–Hochberg correction against clinical data) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Produces the factor values matrix that is correlated with clinical features)
- **R** (Environment in which the correlation/Wilcoxon testing and BH correction are executed (cor.test, wilcox.test, p.adjust))

## Examples

```
Rscript -e 'ct <- cor.test(mofa_factors$Factor1, clinical$Age, method="pearson"); wt <- wilcox.test(mofa_factors$Factor1 ~ clinical$Group); padj <- p.adjust(c(ct$p.value, wt$p.value), method="BH")'
```

## Evaluation signals

- mofa_factor_clinical_correlations.csv exists as a CSV output of the factor-interpretation step
- The output table contains an entry for every tested factor-feature combination (no missing factor-feature pairs)
- A BH-adjusted p-value column is present alongside raw p-values, and the correction was applied across pooled Pearson + Wilcoxon p-values, not per test family
- Numeric clinical features were tested with Pearson correlation and binary features with the Wilcoxon rank-sum test, as recorded in the test-type column
- Discriminant MOFA factors can be ranked by their BH-adjusted association with clinical data

## Limitations

- Correlation alone is insufficient for factor annotation — BiomiX deliberately relies on three methods (clinical correlation, pathway analysis, PubMed bibliography mining), whereas X-omics ACTION only performs correlation analysis
- Earlier versions had a bug where the correlation analysis failed to recognize 'undefined' omics contributors; fixed in BiomiX 2.4, so verify the version used
- Users are expected to carefully review the dataset before running BiomiX; there is no automated validation of input clinical metadata
- Mac installation is still unreliable (use Linux or Windows), which affects reproducibility of the R-based module on some platforms

## Evidence
