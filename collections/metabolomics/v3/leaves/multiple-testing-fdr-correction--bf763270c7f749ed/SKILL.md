---
name: multiple-testing-fdr-correction
description: Use when testing thousands of metabolic features or dozens of KEGG pathways
  for significance (e.g., volcano-plot differential testing with Log2FC > 1.5, or
  hypergeometric MSEA), where uncorrected p-values would yield inflated false positives.
when_to_use_negative:
- When testing a single pre-specified hypothesis or a handful of targeted metabolites
  where multiplicity is negligible
- When the task is exploratory ranking (e.g., PCA, HCA, or batch-effect scoring with
  adjusted R²) rather than hypothesis testing requiring p-value correction
- When the input p-values are already FDR-adjusted; re-applying correction on q-values
  would over-penalize
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: dbnorm
  role: Produces the corrected LC-MS data matrices on which the differential (Log2FC,
    adjusted-p) and regression (Bonferroni) tests are subsequently run
  repo: https://github.com/NBDZ/dbnorm
- name: limma
  role: Used with R stats' lm to fit the CKD-EPI–on–creatinine multiple regression
    and supply p-values for Bonferroni-corrected metabolite selection
- name: R
  role: Environment in which differential testing, regression fitting, and multiple-testing
    correction are performed
- name: ConsensusPathDB
  role: Performs hypergeometric over-representation of the differential metabolite
    list against KEGG pathways and returns FDR-corrected q-values
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:e42a8294daa9505c4560f2a1152caf3b8d749e06577d629fa12b4626d215866f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ComBat
  - ConsensusPathDB
  - Human Metabolome Database (HMDB)
  - KEGG pathways database
  - R
  - ber
  - dbnorm
  - limma
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multiple-testing-fdr-correction

## Summary

Corrects p-values across thousands of metabolomic features or KEGG pathway tests using false discovery rate (q-values) or Bonferroni thresholds so that LC-MS differential and enrichment analyses control the expected false-positive rate.

## When to use

Use when testing thousands of metabolic features or dozens of KEGG pathways for significance (e.g., volcano-plot differential testing with Log2FC > 1.5, or hypergeometric MSEA), where uncorrected p-values would yield inflated false positives.

## When NOT to use

- When testing a single pre-specified hypothesis or a handful of targeted metabolites where multiplicity is negligible
- When the task is exploratory ranking (e.g., PCA, HCA, or batch-effect scoring with adjusted R²) rather than hypothesis testing requiring p-value correction
- When the input p-values are already FDR-adjusted; re-applying correction on q-values would over-penalize

## Inputs

- Vector or table of raw p-values from per-metabolite differential tests (e.g., HFD vs control, untargeted LC-MS features)
- Raw p-values from hypergeometric pathway over-representation tests (ConsensusPathDB vs KEGG pathways)
- Raw p-values from multiple linear regression of CKD-EPI on LC-MS creatinine adjusted for age and sex
- Log2 fold-change vector aligned to the p-value vector for joint thresholding

## Outputs

- Adjusted p-value (q-value) vector per metabolite or pathway
- Filtered list of significant metabolites meeting Log2FC > 1.5 and adjusted p < 0.05
- Significant KEGG pathway table (≥2 metabolites, q < 0.05) with negative log2-transformed q-values
- Bonferroni-corrected regression significance table per metabolite

## How to apply

For feature-level differential analysis (e.g., high-fat-diet vs control in the untargeted LC-MS dataset with 11,156 metabolic profiles), compute per-feature p-values and fold changes, then apply an adjusted p-value (FDR/BH) threshold alongside a Log2FC > 1.5 cutoff to draw the differential metabolite list used in volcano plots. For pathway-level testing, submit the HMDB-filtered metabolite list to ConsensusPathDB against KEGG pathways (4289 compound IDs), where the hypergeometric p-values are FDR-corrected into q-values, keeping pathways with q < 0.05 and ≥2 member metabolites. For regression-based association tests (e.g., CKD-EPI regressed on LC-MS creatinine adjusted for age and sex using lm in R stats/limma), apply Bonferroni correction as the significance criterion for metabolite selection. The rationale: raw p-values across thousands of correlated m/z features dramatically overstate significance, so FDR (q-value) controls the expected proportion of false discoveries while Bonferroni provides the stricter family-wise control the article adopts for regression selection.

## Related tools

- **dbnorm** (Produces the corrected LC-MS data matrices on which the differential (Log2FC, adjusted-p) and regression (Bonferroni) tests are subsequently run) — https://github.com/NBDZ/dbnorm
- **limma** (Used with R stats' lm to fit the CKD-EPI–on–creatinine multiple regression and supply p-values for Bonferroni-corrected metabolite selection)
- **R** (Environment in which differential testing, regression fitting, and multiple-testing correction are performed)
- **ConsensusPathDB** (Performs hypergeometric over-representation of the differential metabolite list against KEGG pathways and returns FDR-corrected q-values)

## Examples

```
fit <- lm(CKD_EPI ~ creatinine + age + sex, data = skpogh); p <- summary(fit)$coefficients["creatinine",4]; p_adj <- p * n_tests  # Bonferroni, or p.adjust(p, method="BH") for FDR
```

## Evaluation signals

- In the SKIPOGH regression, the CKD-EPI–on–creatinine association must be non-significant in raw data but significant after correction with Bonferroni p < 0.05
- The differential metabolite list must contain only features passing BOTH Log2FC > 1.5 and adjusted p-value < 0.05, not p alone
- KEGG enrichment output must include lipid metabolism pathways (alpha-Linolenic acid metabolism, fatty acid biosynthesis) with ≥2 metabolites and q-value < 0.05
- Reported q-values should be ≥ the corresponding raw p-values (monotonicity of BH/FDR correction)
- Output tables (e.g., creatinine_correlation_regression_stats.csv, pathway enrichment CSV) must include explicit corrected p/q-value columns, not just raw p

## Limitations

- FDR correction on very small metabolite subsets (e.g., the raw-data metabolite list) can be so stringent that no pathway reaches q < 0.05, masking real lipid-metabolism signals
- Bonferroni is stricter than BH-FDR and may under-detect weak regression associations; the article applies it specifically for metabolite selection in the SKIPOGH regression
- Residual batch association in lowess- or nonparametric ComBat-corrected data (up to 60–78% residual variability) can distort p-values upstream of any multiple-testing correction

## Evidence

- [results] Volcano plot differential thresholds: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [other] MSEA q-value threshold: [evidence span withheld — anchor status: not_found]
- [other] Bonferroni in regression: [evidence span withheld — anchor status: not_found]
- [results] Regulation scope for correction: "Unsupervised multivariate analysis of 11,156 metabolic profiles (i.e., PCA and Hierarchical Cluster Analysis (HCA)) performed with “dbnorm”"
- [readme] README correction-tool scope: "the performance of applied models are presented by a score calculated based on the maximum variability explained by the batch level"
