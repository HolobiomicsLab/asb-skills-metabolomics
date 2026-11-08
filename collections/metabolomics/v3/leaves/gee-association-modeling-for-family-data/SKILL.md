---
name: gee-association-modeling-for-family-data
description: Use when testing whether latent omics factors (e.g., MOFA factors from
  buccal DNA methylation and urine metabolomics) or cluster memberships relate to
  behavioral scores in family/twin data requiring relatedness-adjusted regression.
when_to_use_negative:
- Subjects are independent (no family relatedness) — ordinary regression or simple
  correlation is sufficient and GEE's exchangeable correlation is unnecessary
- Factors have not yet been derived — MOFA factor extraction, SNF clustering, and
  MCA must be run first; this skill only tests associations between already-computed
  factors
- The goal is multi-omics integration or clustering rather than statistical association
  testing of factors against behavioral dimensions
tools:
- name: Nextflow
  role: Workflow manager that packages and runs the modular association-analysis step
    within the ACTIONdemonstrator pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime pulling the software dependencies for the workflow's association-testing
    processes
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:f6b81f5a2f500a7ca6afd6e73e1c4b42ea7b40b27616032f9e4795ff533e1707
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - Nextflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gee-association-modeling-for-family-data

## Summary

Associates MOFA omics factors and SNF cluster memberships with MCA behavioral dimension scores in the ACTION Biomarker case study, using GEE models with exchangeable working correlation to handle family relatedness in the NTR twin cohort and non-parametric/correlation tests in the LUMC-Curium cohort, with FDR correction across all tested factor–dimension pairs.

## When to use

Use when testing whether latent omics factors (e.g., MOFA factors from buccal DNA methylation and urine metabolomics) or cluster memberships relate to behavioral scores in family/twin data requiring relatedness-adjusted regression.

## When NOT to use

- Subjects are independent (no family relatedness) — ordinary regression or simple correlation is sufficient and GEE's exchangeable correlation is unnecessary
- Factors have not yet been derived — MOFA factor extraction, SNF clustering, and MCA must be run first; this skill only tests associations between already-computed factors
- The goal is multi-omics integration or clustering rather than statistical association testing of factors against behavioral dimensions

## Inputs

- MOFA omics factors derived from buccal DNA methylation and urine metabolomics
- SNF cluster memberships from SNFpy with cluster count chosen by the eigengap method
- MCA behavioral dimension scores computed with FactoMineR
- Cohort assignment / family-relatedness identifiers for the NTR twin cohort

## Outputs

- association_statistics.csv listing cohort, predictor (omics factor or cluster membership), behavioral dimension, test type, test statistic, raw p-value, and FDR-corrected p-value per tested pair

## How to apply

Load the MOFA factors (MOFA2), SNF cluster memberships (cluster count chosen by the eigengap method in SNFpy), and MCA behavioral dimension scores computed with FactoMineR. For the NTR twin cohort, fit generalized estimating equation models with the R package 'gee' using an exchangeable working correlation structure and Gaussian link, regressing each behavioral dimension score on each MOFA factor and SNF cluster membership, with up to 100 iterations per model. For the LUMC-Curium cohort of children, use Mann–Whitney U tests for 2 SNF clusters or Kruskal–Wallis tests for >2 clusters, and Spearman's rank correlation among omics and behavioral factors. Correct all p-values with FDR across the tested factor–dimension pairs and assemble association_statistics.csv with cohort, predictor, behavioral dimension, test type, test statistic, and raw plus FDR-corrected p-values.

## Related tools

- **Nextflow** (Workflow manager that packages and runs the modular association-analysis step within the ACTIONdemonstrator pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime pulling the software dependencies for the workflow's association-testing processes) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --behavioral_data Synthetic_data/synthetic_cbcl_data.csv
```

## Evaluation signals

- association_statistics.csv contains exactly one row per tested factor–dimension combination for both cohorts (NTR and LUMC-Curium)
- Each NTR row records GEE with exchangeable working correlation structure and Gaussian link as the test type
- Each LUMC-Curium row records the correct non-parametric test: Mann–Whitney U for 2 clusters, Kruskal–Wallis for more than 2
- Every row includes both a raw and an FDR-corrected p-value
- NTR models complete within the maximum of 100 GEE iterations per model

## Limitations

- Privacy regulations prohibit sharing the real NTR and LUMC-Curium omics/clinical data, so only synthetic data is distributed for demonstrating this step
- GEE models for related twins can require many iterations (capped at 100 per model) and may not converge for some factor–dimension pairs
- The correct test choice depends on cluster count: a miscounted SNF clustering (eigengap) leads to the wrong Curium test (Mann–Whitney vs. Kruskal–Wallis)

## Evidence

- [discussion] We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [discussion] We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman's rank correlation: "We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman’s rank correlation"
- [readme] nextflow run action.nf --behavioral_data Synthetic_data/synthetic_cbcl_data.csv --phenotype_covariates Synthetic_data/synthetic_phenotype_covariates_data.csv: "nextflow run action.nf 
            --output dir/of/choice
            --mtblmcs_values Sythetic_data/synthetic_metabolomics.csv
            --maf_files Synthetic_data/"
- [abstract] The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies.: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
