---
name: pleiotropy-exclusion-in-mr
description: Use when integrating an mGWAS summary statistics exposure (e.g. L-Cystathionine)
  with a GWAS outcome (e.g. finn-b-E4_DM2) in MetaboAnalyst 6.0 and you must exclude
  invalid instruments for pleiotropy before MR estimation.
when_to_use_negative:
- If you are only doing standard metabolomics statistical analysis of peak intensities
  (e.g. multi-factor designs) and not estimating a metabolite-to-phenotype causal
  relation.
- If you have no mGWAS summary statistics available for the metabolite exposure; MR
  cannot proceed without genome-wide SNP associations for the exposure.
- If the task is raw MS spectra processing, MS2 annotation, or dose-response benchmark
  dose calculation — these are different MetaboAnalyst 6.0 modules.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3337
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalyst 6.0
  role: Causal Analysis module performing two-sample Mendelian randomization, including
    instrument selection, LD clumping, harmonization and pleiotropy exclusion
- name: MetaboAnalystR 4.0
  role: R functions and libraries underlying the MetaboAnalyst web server, enabling
    local reproduction of the same Causal Analysis workflow
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OpenGWAS
  role: Source of the GWAS outcome summary statistics (e.g. finn-b-E4_DM2) and 1000
    Genomes LD reference for clumping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:76d0f65e9f435e152e35eeb85e7a8d033c9ebef741c54bfc8eefafa83e53f5d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  - OpenGWAS
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3337
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pleiotropy-exclusion-in-mr

## Summary

Applies MetaboAnalyst 6.0's Causal Analysis module (two-sample Mendelian randomization) pleiotropy/invalid-instrument exclusion before estimating a metabolite-to-phenotype causal effect, ensuring only valid instruments remain for MR estimation.

## When to use

Use when integrating an mGWAS summary statistics exposure (e.g. L-Cystathionine) with a GWAS outcome (e.g. finn-b-E4_DM2) in MetaboAnalyst 6.0 and you must exclude invalid instruments for pleiotropy before MR estimation.

## When NOT to use

- If you are only doing standard metabolomics statistical analysis of peak intensities (e.g. multi-factor designs) and not estimating a metabolite-to-phenotype causal relation.
- If you have no mGWAS summary statistics available for the metabolite exposure; MR cannot proceed without genome-wide SNP associations for the exposure.
- If the task is raw MS spectra processing, MS2 annotation, or dose-response benchmark dose calculation — these are different MetaboAnalyst 6.0 modules.

## Inputs

- mGWAS summary statistics for the metabolite exposure (e.g. L-Cystathionine)
- GWAS outcome summary statistics (e.g. finn-b-E4_DM2 type 2 diabetes, retrieved from OpenGWAS)
- Selected instruments (SNPs) with harmonized effect alleles

## Outputs

- Filtered set of valid, non-pleiotropic SNP instruments
- MR results table (causal effect estimate, standard error, p-value, method) saved as mr_cystathionine_e4_dm2_results.csv

## How to apply

First load the metabolite mGWAS summary statistics as the exposure and the corresponding GWAS outcome (e.g. finn-b-E4_DM2 type 2 diabetes) in MetaboAnalyst 6.0's Causal Analysis module. Select SNPs associated with the exposure using the module's instrument selection with default significance thresholds, then LD-clump them on the OpenGWAS 1000 Genomes LD reference to remove correlated variants. Harmonize effect alleles and directions between exposure and outcome. Next, apply the module's default pleiotropy exclusion (e.g. Steiger filtering and invalid-instrument/outlier removal) to retain only valid, non-pleiotropic instruments before running the inverse-variance weighted MR estimator; this step is the causal-inference rationale that ensures the SNP-outcome association is mediated by the exposure rather than horizontal pleiotropy, yielding a defensible causal effect estimate.

## Related tools

- **MetaboAnalyst 6.0** (Causal Analysis module performing two-sample Mendelian randomization, including instrument selection, LD clumping, harmonization and pleiotropy exclusion)
- **MetaboAnalystR 4.0** (R functions and libraries underlying the MetaboAnalyst web server, enabling local reproduction of the same Causal Analysis workflow) — https://github.com/xia-lab/MetaboAnalystR
- **OpenGWAS** (Source of the GWAS outcome summary statistics (e.g. finn-b-E4_DM2) and 1000 Genomes LD reference for clumping)

## Examples

```
MetaboAnalyst 6.0 web: Causal Analysis module → load L-Cystathionine mGWAS exposure + finn-b-E4_DM2 GWAS outcome from OpenGWAS → Search instruments → LD clump → Harmonize → apply default pleiotropy exclusion → Estimate causal effect → download mr_cystathionine_e4_dm2_results.csv
```

## Evaluation signals

- The number of instruments retained after pleiotropy exclusion should be smaller than the number of clumped/harmonized SNPs, and the pruned SNPs should be recorded as excluded due to pleiotropy or invalidity.
- The final MR results table must contain columns for the causal effect estimate, standard error, p-value, and the MR method used (inverse-variance weighted plus sensitivity estimators).
- Re-running the workflow on the same mGWAS exposure and GWAS outcome should yield the same set of retained instruments and the same effect estimate, demonstrating reproducibility from the module's default settings.
- The excluded instruments should show signs of horizontal pleiotropy or invalidity (e.g. Steiger-direction mismatch or outlier status), consistent with the module's default pleiotropy-removal criteria.
- The causal effect direction and significance should be consistent with published results (e.g. altered cystathionine level showing a causal effect on type 2 diabetes).

## Limitations

- The module's pleiotropy exclusion depends on default significance and clumping parameters; deviations from defaults may change which instruments are retained and thus the causal estimate.
- As with all two-sample MR, vertical pleiotropy assumptions require that the SNP-outcome association is mediated by the exposure; hidden or dynamic pleiotropy can bias the causal estimate.
- The LD clumping draws on OpenGWAS 1000 Genomes reference data, so results depend on that reference population's LD structure.

## Evidence

- [abstract] Causal analysis of metabolite-phenotype relations via two-sample Mendelian randomization: "a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization"
- [intro] Causal relationships between a metabolite and a phenotype of interest can be estimated by integrating mGWAS data with comparable GWAS data through Mendelian randomization: "By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest"
- [readme] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation.: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server"
- [readme] MetaboAnalystR is synchronized with the web server. After installing and loading the package, users will be able to reproduce the same results from their local computers: "The package is synchronized with the MetaboAnalyst web server. After installing and loading the package, users will be able to reproduce the same results from their local computers"
