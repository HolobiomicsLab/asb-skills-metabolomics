---
name: two-sample-mr-causal-effect-estimation
description: Use when an agent must test whether altered metabolite levels causally
  affect a phenotype by combining mGWAS summary statistics for a metabolite with a
  GWAS dataset (e.g. OpenGWAS ID finn-b-E4_DM2) via two-sample MR.
when_to_use_negative:
- Input is raw MS spectra or a metabolite feature table — use the Spectra Processing
  or Statistical Analysis modules of MetaboAnalyst 6.0 instead (this skill addresses
  causal analysis, not peak detection/annotation or multi-factor statistical testing).
- The task is dose-response risk assessment of chemical exposures — the Dose-Response
  Analysis module with benchmark dose calculations is the correct module, not two-sample
  MR.
- No GWAS/mGWAS summary statistics are available (e.g. only cohort-level metabolite
  measurements), since MR requires genotype-metabolite and genotype-phenotype associations
  to define instruments.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: MetaboAnalyst 6.0
  role: Web platform providing the Causal Analysis module used to run instrument selection,
    LD clumping, harmonization, pleiotropy exclusion, and IVW/sensitivity MR estimation
    under default parameters.
- name: MetaboAnalystR 4.0
  role: R package containing the functions underlying the MetaboAnalyst web server,
    enabling local reproduction of the same MR workflow and results via downloaded
    R command history.
  repo: https://github.com/xia-lab/MetaboAnalystR.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:1e8622b60e70eda4a3e64ff8c79dcdff7138296717bc2dd2985b92a94a4afeae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR.git
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR 4.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# two-sample-mr-causal-effect-estimation

## Summary

Estimates causal relationships between a metabolite (exposure) and a phenotype/disease (outcome) by two-sample Mendelian randomization, using the MetaboAnalyst 6.0 Causal Analysis module to integrate metabolite GWAS (mGWAS) summary statistics with phenotype GWAS data.

## When to use

Use when an agent must test whether altered metabolite levels causally affect a phenotype by combining mGWAS summary statistics for a metabolite with a GWAS dataset (e.g. OpenGWAS ID finn-b-E4_DM2) via two-sample MR.

## When NOT to use

- Input is raw MS spectra or a metabolite feature table — use the Spectra Processing or Statistical Analysis modules of MetaboAnalyst 6.0 instead (this skill addresses causal analysis, not peak detection/annotation or multi-factor statistical testing).
- The task is dose-response risk assessment of chemical exposures — the Dose-Response Analysis module with benchmark dose calculations is the correct module, not two-sample MR.
- No GWAS/mGWAS summary statistics are available (e.g. only cohort-level metabolite measurements), since MR requires genotype-metabolite and genotype-phenotype associations to define instruments.

## Inputs

- mGWAS summary statistics for the metabolite of interest (exposure, e.g. L-Cystathionine)
- GWAS summary statistics for the phenotype/disease from OpenGWAS (outcome, e.g. finn-b-E4_DM2 for type 2 diabetes)
- Default module parameters: SNP-significance thresholds, LD clumping r^2 and window parameters, pleiotropy-exclusion settings

## Outputs

- MR results table (CSV, e.g. mr_cystathionine_e4_dm2_results.csv) with causal effect estimate, standard error, p-value, and method columns
- Set of retained valid genetic instruments after LD clumping, harmonization, and pleiotropy exclusion

## How to apply

Load the metabolite mGWAS summary statistics as the exposure and fetch the outcome GWAS summary statistics from OpenGWAS (e.g. finn-b-E4_DM2 for type 2 diabetes). Use the Causal Analysis module's instrument selection with default significance thresholds to find SNPs associated with the exposure, then perform LD clumping on the selected instruments with default r^2 and window parameters using OpenGWAS 1000 Genomes LD reference data to remove correlated variants. Harmonize the exposure and outcome effect alleles/directions for the clumped instruments, and apply default pleiotropy exclusion (Steiger filtering / outlier and invalid-instrument removal) to retain valid instruments. Estimate the causal effect with the module's default MR estimation methods — inverse-variance weighted plus sensitivity estimators — and save the results table (causal effect estimate, standard error, p-value, method columns) as a CSV such as mr_cystathionine_e4_dm2_results.csv.

## Related tools

- **MetaboAnalyst 6.0** (Web platform providing the Causal Analysis module used to run instrument selection, LD clumping, harmonization, pleiotropy exclusion, and IVW/sensitivity MR estimation under default parameters.)
- **MetaboAnalystR 4.0** (R package containing the functions underlying the MetaboAnalyst web server, enabling local reproduction of the same MR workflow and results via downloaded R command history.) — https://github.com/xia-lab/MetaboAnalystR.git

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- The saved MR results table contains the expected columns: causal effect estimate, standard error, p-value, and method.
- The inverse-variance weighted method appears in the results alongside sensitivity estimators, with a p-value indicating whether the metabolite-phenotype causal effect is significant.
- The instrument set is reduced after LD clumping (correlated variants removed at default r^2/window) and after pleiotropy exclusion (Steiger filtering / outlier removal), i.e. no LD-correlated or invalid instruments remain.
- Exposure and outcome effect alleles/directions are harmonized before estimation (no allele-flip inconsistencies in the harmonized table).
- The causal conclusion is biologically consistent with prior literature — e.g. default-parameter analysis of L-Cystathionine against finn-b-E4_DM2 found a causal effect on type 2 diabetes consistent with a recent study.

## Limitations

- Validity depends on MR assumptions; the workflow relies on default pleiotropy exclusion (Steiger filtering / outlier and invalid-instrument removal), and default significance/clumping thresholds may not suit every exposure-outcome pair.
- Requires access to OpenGWAS summary statistics and 1000 Genomes LD reference data for outcome data and clumping.
- The paper notes future-work gaps such as better integration with other omics data, and no changelog was found for the MetaboAnalystR repository, so version-specific MR behaviour may be undocumented.

## Evidence

- [abstract] Causal analysis module purpose: "a Causal Analysis module for estimating metabolite - phenotype causal relations based on two-sample Mendelian randomization"
- [intro] Rationale for integrating mGWAS and GWAS: "By integrating mGWAS data with comparable GWAS data that associate genotypes with various phenotypes (17), we can now estimate causal relationships between a metabolite and a phenotype of interest"
- [other] Task-card result under default parameters: [evidence span withheld — anchor status: not_found]
- [readme] R package enabling local reproduction: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
