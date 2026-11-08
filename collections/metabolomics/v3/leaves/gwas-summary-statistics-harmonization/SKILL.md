---
name: gwas-summary-statistics-harmonization
description: Use when estimating causal relationships between a metabolite (from mGWAS
  summary statistics) and a phenotype (from comparable GWAS data) via two-sample Mendelian
  randomization, e.g. testing L-Cystathionine against type 2 diabetes.
when_to_use_negative:
- Input is already a harmonized and LD-clumped instrument set ready for MR estimation
  — re-running harmonization risks removing valid instruments.
- Analysis goal is spectral processing, MS2 annotation, statistical analysis of complex
  designs, or dose-response/benchmark dose calculations — these are distinct MetaboAnalyst
  6.0 modules, not Causal Analysis.
- No mGWAS exposure summary statistics are available (only metabolite concentrations),
  since two-sample MR requires instrument SNPs from a metabolite GWAS.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0154
tools:
- name: MetaboAnalyst 6.0
  role: Web platform whose Causal Analysis module performs SNP instrument selection,
    LD clumping, harmonization, pleiotropy exclusion, and MR estimation
- name: OpenGWAS
  role: Source of outcome GWAS summary statistics (e.g. finn-b-E4_DM2) and 1000 Genomes
    LD reference data used for clumping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:a976be66942ad12d79cf793f9074e231ad46193acd9f09ecf4cca37e4807591e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 6.0
  - OpenGWAS
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gwas-summary-statistics-harmonization

## Summary

Harmonize GWAS/mGWAS summary statistics (effect alleles, signs, and instrument SNP sets) from exposure and outcome datasets within MetaboAnalyst 6.0's Causal Analysis module so that two-sample Mendelian randomization can estimate metabolite–phenotype causal effects.

## When to use

Use when estimating causal relationships between a metabolite (from mGWAS summary statistics) and a phenotype (from comparable GWAS data) via two-sample Mendelian randomization, e.g. testing L-Cystathionine against type 2 diabetes.

## When NOT to use

- Input is already a harmonized and LD-clumped instrument set ready for MR estimation — re-running harmonization risks removing valid instruments.
- Analysis goal is spectral processing, MS2 annotation, statistical analysis of complex designs, or dose-response/benchmark dose calculations — these are distinct MetaboAnalyst 6.0 modules, not Causal Analysis.
- No mGWAS exposure summary statistics are available (only metabolite concentrations), since two-sample MR requires instrument SNPs from a metabolite GWAS.

## Inputs

- Exposure mGWAS summary statistics (e.g. GWAS summary statistics for L-Cystathionine)
- Outcome GWAS summary statistics from OpenGWAS (e.g. finn-b-E4_DM2 for type 2 diabetes)
- 1000 Genomes LD reference data (via OpenGWAS) for clumping

## Outputs

- Harmonized effect-allele-aligned SNP table for the selected instruments
- MR results table (mr_cystathionine_e4_dm2_results.csv) with columns: causal effect estimate, standard error, p-value, and method

## How to apply

Load the metabolite GWAS (mGWAS) summary statistics for the exposure trait and fetch the outcome GWAS summary statistics from OpenGWAS (e.g. finn-b-E4_DM2 for type 2 diabetes). In MetaboAnalyst 6.0's Causal Analysis module, select instruments (SNPs) associated with the exposure using the module's default significance thresholds, then perform LD clumping on those instruments using the module's default r² and window parameters with the OpenGWAS 1000 Genomes LD reference to remove correlated variants. Next, harmonize the exposure and outcome effect alleles and effect directions for the clumped instrument SNPs, and apply the module's default pleiotropy exclusion (e.g. Steiger filtering and outlier/invalid-instrument removal) to retain valid instruments. Finally, estimate the causal effect with the module's default MR estimators (inverse-variance weighted plus sensitivity methods) and write the results table. Rationale: the article states that integrating mGWAS data with comparable GWAS data through Mendelian randomization allows estimation of causal relationships between a metabolite and a phenotype, and harmonization is a prerequisite step before the IVW estimator and sensitivity analyses can be trusted.

## Related tools

- **MetaboAnalyst 6.0** (Web platform whose Causal Analysis module performs SNP instrument selection, LD clumping, harmonization, pleiotropy exclusion, and MR estimation)
- **OpenGWAS** (Source of outcome GWAS summary statistics (e.g. finn-b-E4_DM2) and 1000 Genomes LD reference data used for clumping)

## Examples

```
MetaboAnalyst 6.0 Causal Analysis: load mGWAS exposure stats for L-Cystathionine, fetch finn-b-E4_DM2 from OpenGWAS, run default SNP filtering, LD clumping, harmonization, and MR to produce mr_cystathionine_e4_dm2_results.csv
```

## Evaluation signals

- Harmonized table has zero mismatched effect-allele rows and consistent sign alignment between exposure and outcome effects for each clumped SNP.
- Instrument SNP count is reduced after LD clumping (no remaining pairs above the module's default r² threshold in the default window).
- Pleiotropy/Steiger filtering removes invalid instruments and the retained set yields an inverse-variance-weighted estimate with a reported p-value and standard error.
- Saved results CSV contains the columns 'causal effect estimate', 'standard error', 'p-value', and 'method' and reproduces the reported finding that altered cystathionine level has a causal effect on type 2 diabetes.
- Running the module with default parameters reproduces the workflow documented for the L-Cystathionine / finn-b-E4_DM2 example.

## Limitations

- Harmonization and MR estimation depend on the module's default pleiotropy exclusion (e.g. Steiger filtering, outlier and invalid-instrument removal); misconfigured filtering can bias the causal estimate.
- Clumping accuracy depends on the LD reference; using the OpenGWAS 1000 Genomes reference may not match all ancestries present in the mGWAS exposure data.
- The workflow requires access to OpenGWAS summary statistics; unavailable IDs (e.g. finn-b-E4_DM2) cannot be harmonized.

## Evidence

- [abstract] a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization: "a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization"
- [intro] By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes, we can now estimate causal relationships between a metabolite and a phenotype of interest: "By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest"
- [other] Using default parameters for SNP filtering, harmonization, and MR analysis, the module found that a significantly altered cystathionine level has a causal effect on type 2 diabetes: [evidence span withheld — anchor status: not_found]
- [other] Perform LD clumping on the selected instruments using default r^2 and window parameters to remove correlated variants, drawing on OpenGWAS 1000 Genomes LD reference data.: [evidence span withheld — anchor status: not_found]
