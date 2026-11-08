---
name: mendelian-randomization-instrument-selection-from-mgwas
description: Use when estimating a causal relationship between a metabolite and a
  phenotype by integrating mGWAS data with comparable phenotype GWAS data via two-sample
  Mendelian randomization.
when_to_use_negative:
- Input is already a feature table or raw MS spectra — this skill operates on GWAS/mGWAS
  summary statistics, not metabolomics peak tables
- The research goal is metabolite identification or spectral annotation (use the MS2
  Peak Annotation module instead), not metabolite-phenotype causal inference
- No comparable GWAS summary statistics exist for the phenotype of interest, since
  the module integrates mGWAS data with comparable GWAS data
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_3399
tools:
- name: MetaboAnalyst 6.0
  role: Web platform whose Causal Analysis module performs instrument search, LD clumping,
    harmonization, pleiotropy exclusion, and two-sample MR estimation
- name: MetaboAnalystR
  role: R package synchronized with the MetaboAnalyst web server enabling local reproduction
    of the same analysis from the R command history downloaded from MetaboAnalyst
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: asari
  role: Not used for this skill (it is for LC-MS spectra processing); listed to clarify
    scope boundaries
  repo: https://github.com/xia-lab/OptiLCMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:1c9466a2b78cafcdf7196087fa127f537aef75be892c1e1414dd70b8b68d7a8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - asari
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mendelian-randomization-instrument-selection-from-mgwas

## Summary

Select SNP instruments for a metabolite exposure from mGWAS summary statistics and run two-sample Mendelian randomization against a phenotype GWAS (e.g. finn-b-E4_DM2) to estimate metabolite-phenotype causal effects using MetaboAnalyst 6.0's Causal Analysis module.

## When to use

Use when estimating a causal relationship between a metabolite and a phenotype by integrating mGWAS data with comparable phenotype GWAS data via two-sample Mendelian randomization.

## When NOT to use

- Input is already a feature table or raw MS spectra — this skill operates on GWAS/mGWAS summary statistics, not metabolomics peak tables
- The research goal is metabolite identification or spectral annotation (use the MS2 Peak Annotation module instead), not metabolite-phenotype causal inference
- No comparable GWAS summary statistics exist for the phenotype of interest, since the module integrates mGWAS data with comparable GWAS data

## Inputs

- mGWAS summary statistics for the metabolite of interest (exposure, e.g. L-Cystathionine)
- Phenotype GWAS summary statistics fetched from OpenGWAS (outcome, e.g. GWAS ID finn-b-E4_DM2)
- OpenGWAS 1000 Genomes LD reference data used for clumping

## Outputs

- MR results table with causal effect estimate, standard error, p-value, and method columns (saved as e.g. mr_cystathionine_e4_dm2_results.csv)
- Selected, LD-clumped, and harmonized SNP instrument set

## How to apply

Load mGWAS summary statistics for the metabolite (e.g. L-Cystathionine) as the exposure and fetch the phenotype GWAS summary statistics (e.g. finn-b-E4_DM2 from OpenGWAS) as the outcome into the Causal Analysis module. Search for SNPs associated with the exposure using the module's instrument selection with default significance thresholds, then LD-clump the selected instruments against the OpenGWAS 1000 Genomes LD reference using default r^2 and window parameters to remove correlated variants. Harmonize exposure and outcome effect alleles/directions for the clumped instruments, and apply the module's default pleiotropy exclusion (Steiger filtering / outlier and invalid-instrument removal) to retain valid instruments. Estimate the causal effect using the module's default MR estimators (inverse-variance weighted plus sensitivity estimators); this workflow reproduced a published causal effect of altered cystathionine level on type 2 diabetes using default parameters throughout, so avoid non-default tweaks unless justified.

## Related tools

- **MetaboAnalyst 6.0** (Web platform whose Causal Analysis module performs instrument search, LD clumping, harmonization, pleiotropy exclusion, and two-sample MR estimation)
- **MetaboAnalystR** (R package synchronized with the MetaboAnalyst web server enabling local reproduction of the same analysis from the R command history downloaded from MetaboAnalyst) — https://github.com/xia-lab/MetaboAnalystR
- **asari** (Not used for this skill (it is for LC-MS spectra processing); listed to clarify scope boundaries) — https://github.com/xia-lab/OptiLCMS

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- MR results CSV contains columns for causal effect estimate, standard error, p-value, and MR method
- Causal direction is significant (default thresholds) for a known validated relation, e.g. cystathionine -> type 2 diabetes, consistent with a recent study
- Instrument count is reduced after LD clumping relative to the initial instrument search, indicating correlated variants were removed
- Harmonization step resolves allele mismatches so effect alleles/directions are consistent between exposure and outcome
- All steps (instrument selection, clumping, harmonization, pleiotropy exclusion, MR estimation) can be re-run with default parameters via the R command history downloaded from the MetaboAnalyst web server

## Limitations

- Default pleiotropy exclusion is limited to Steiger filtering / outlier and invalid-instrument removal; residual horizontal pleiotropy may remain if these checks pass
- MR requires comparable GWAS data for both metabolite and phenotype; results are estimates of causal relationships, not proofs
- MetaboAnalystR is 'freely available from GitHub' but the main repo license shows NOASSERTION, so reuse of code for derivative work requires checking the license directly

## Evidence

- [abstract] a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization: "a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization"
- [intro] By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest: "By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest"
- [readme] devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T): "devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)"
- [readme] The package is synchronized with the MetaboAnalyst web server. After installing and loading the package, users will be able to reproduce the same results from their local computers using the corresponding R command history downloaded from MetaboAnalyst web site: "The package is synchronized with the MetaboAnalyst web server. After installing and loading the package, users will be able to reproduce the same results from their local computers using the"
