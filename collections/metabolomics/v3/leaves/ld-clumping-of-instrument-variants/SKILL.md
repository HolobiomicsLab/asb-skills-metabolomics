---
name: ld-clumping-of-instrument-variants
description: Use when running a two-sample Mendelian randomization analysis (e.g.
  an mGWAS metabolite as exposure against a GWAS outcome such as finn-b-E4_DM2) and
  the selected instrument SNPs need pruning of correlated variants before harmonization.
when_to_use_negative:
- When the input is already a pruned, LD-independent variant list and no correlation
  structure exists among the instrument SNPs
- When the analysis goal is raw mGWAS/GWAS association testing rather than Mendelian
  randomization — clumping is only relevant for constructing valid MR instrument sets
- When instrument variants come from a single reference panel with no LD structure
  to account for (e.g. rare independent variants already selected by significance
  threshold alone)
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3393
tools:
- name: MetaboAnalyst 6.0
  role: Causal Analysis module that performs LD clumping of instrument SNPs using
    OpenGWAS 1000 Genomes LD reference data with default r^2 and window parameters
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalystR 4.0
  role: Underlying R package exposing the same Causal Analysis functionality, including
    LD clumping of mGWAS/GWAS instruments
  repo: https://github.com/xia-lab/MetaboAnalystR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:8839755a94715eb49424f023596a5dcd4089d3696992f768f0f99ee110f16ebf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# LD clumping of instrument variants

## Summary

Removes correlated SNPs from the instrument set selected for a metabolite–phenotype Mendelian randomization analysis, so that only linkage-disequilibrium-independent variants contribute to the two-sample MR effect estimate in the MetaboAnalyst 6.0 Causal Analysis module.

## When to use

Use when running a two-sample Mendelian randomization analysis (e.g. an mGWAS metabolite as exposure against a GWAS outcome such as finn-b-E4_DM2) and the selected instrument SNPs need pruning of correlated variants before harmonization.

## When NOT to use

- When the input is already a pruned, LD-independent variant list and no correlation structure exists among the instrument SNPs
- When the analysis goal is raw mGWAS/GWAS association testing rather than Mendelian randomization — clumping is only relevant for constructing valid MR instrument sets
- When instrument variants come from a single reference panel with no LD structure to account for (e.g. rare independent variants already selected by significance threshold alone)

## Inputs

- List of SNP instruments selected from metabolite (mGWAS) summary statistics for the exposure
- mGWAS summary statistics for the metabolite (e.g. L-Cystathionine)
- GWAS summary statistics for the outcome phenotype (e.g. finn-b-E4_DM2) fetched from OpenGWAS
- 1000 Genomes LD reference data accessed via OpenGWAS

## Outputs

- Pruned, LD-independent instrument SNP set used for harmonization and MR estimation
- MR results table (mr_cystathionine_e4_dm2_results.csv) with causal effect estimate, standard error, p-value, and method columns downstream of clumping

## How to apply

After the Causal Analysis module selects SNP instruments associated with the metabolite exposure from the mGWAS summary statistics, pass the instrument list through LD clumping to eliminate variants in linkage disequilibrium with one another. Clumping draws on 1000 Genomes LD reference data accessed via OpenGWAS, using the module's default r^2 threshold and window parameters to group and prune correlated SNPs. The rationale is that correlated instruments can bias the inverse-variance-weighted MR estimate and inflate pleiotropy-related false signals, so only LD-independent variants should be carried forward into harmonization, pleiotropy exclusion (e.g. Steiger filtering), and MR estimation. The clumped instrument set is then harmonized between exposure and outcome effect alleles before the module's default MR estimators are applied.

## Related tools

- **MetaboAnalyst 6.0** (Causal Analysis module that performs LD clumping of instrument SNPs using OpenGWAS 1000 Genomes LD reference data with default r^2 and window parameters) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalystR 4.0** (Underlying R package exposing the same Causal Analysis functionality, including LD clumping of mGWAS/GWAS instruments) — https://github.com/xia-lab/MetaboAnalystR

## Evaluation signals

- After clumping, no pair of retained instrument SNPs should exceed the module's default r^2 threshold within the clumping window (1000 Genomes LD reference)
- The number of SNPs after clumping must be ≤ the number of instruments originally selected by the significance threshold
- Downstream MR results table (mr_cystathionine_e4_dm2_results.csv) should contain the effect estimate, standard error, p-value, and method columns populated from the clumped instrument set
- Re-running clumping with the same input instrument list and default parameters should produce an identical pruned SNP set (deterministic output)
- Harmonized exposure/outcome effect alleles should align in direction for every SNP retained after clumping

## Limitations

- Clumping parameters are default-based; if the default r^2 threshold is too permissive or restrictive for a given metabolite's instrument architecture, results may retain correlated variants or over-prune informative instruments
- LD clumping depends on the 1000 Genomes reference panel ancestry; mismatched ancestry between the reference LD matrix and the mGWAS/GWAS populations can misclassify LD structure
- MetaboAnalyst 6.0's authors note that metabolome annotations still need improvement and integration with other omics data is incomplete, indicating the broader platform remains in flux

## Evidence

- [abstract] a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization: "a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization"
- [intro] By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest: "By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest"
- [other] Perform LD clumping on the selected instruments using default r^2 and window parameters to remove correlated variants, drawing on OpenGWAS 1000 Genomes LD reference data: [evidence span withheld — anchor status: not_found]
- [methods] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
