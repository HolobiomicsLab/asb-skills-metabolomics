---
name: mca-behavioral-factor-association-analysis
description: Use when you have latent factors from multi-omics data (e.g. MOFA factors),
  sample cluster memberships (e.g. SNF clusters), and behavioral dimension scores
  from MCA, and need cohort-aware association testing with multiple-testing correction.
when_to_use_negative:
- Samples are unrelated individuals only in a single cohort — plain regression or
  correlation without a GEE family-correlation structure is sufficient and simpler
- Factors have not yet been computed (no MOFA factors, SNF clusters, or MCA dimensions
  available) — run the factor extraction and clustering workflow steps first
- The association of interest involves raw omics features rather than latent factors
  — this skill tests precomputed MOFA factors and cluster memberships, not individual
  probes or metabolites
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3517
tools:
- name: gee (R package)
  role: Fits Generalized Estimating Equation models with exchangeable working correlation
    and Gaussian link to test associations in the NTR twin cohort while accounting
    for family relatedness
- name: SNFpy
  role: Provides the cluster memberships (optimal cluster number via eigengap method)
    tested as predictors of behavioral dimensions
  repo: https://github.com/Xomics/Docker_containers.git
- name: FactoMineR
  role: Supplies the MCA behavioral dimension scores used as response variables in
    the association models
- name: MOFA2
  role: Supplies the omics factors from buccal DNA methylation and urine metabolomics
    used as predictors
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
- name: Nextflow
  role: Workflow manager that orchestrates the association-analysis module of the
    modular pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow.git
- name: Docker
  role: Containers encapsulate the software dependencies (R packages, Python SNFpy)
    used by the association-testing processes
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:e4faa9ab1c9fe9382bb34b8636ea6eabcdda0474ed2db7f3b723a24edbb6b7f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/Docker_containers.git
  tools:
  - Docker
  - FactoMineR
  - MOFA2
  - Nextflow
  - SNFpy
  - gee (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mca-behavioral-factor-association-analysis

## Summary

Tests whether MOFA omics factors (from buccal DNA methylation and urine metabolomics) and SNF cluster memberships are associated with MCA behavioral dimension scores, using GEE models in the NTR twin cohort (to adjust for family relatedness) and non-parametric tests plus Spearman correlations in the LUMC-Curium cohort, with FDR correction across all tested factor–dimension pairs.

## When to use

Use when you have latent factors from multi-omics data (e.g. MOFA factors), sample cluster memberships (e.g. SNF clusters), and behavioral dimension scores from MCA, and need cohort-aware association testing with multiple-testing correction.

## When NOT to use

- Samples are unrelated individuals only in a single cohort — plain regression or correlation without a GEE family-correlation structure is sufficient and simpler
- Factors have not yet been computed (no MOFA factors, SNF clusters, or MCA dimensions available) — run the factor extraction and clustering workflow steps first
- The association of interest involves raw omics features rather than latent factors — this skill tests precomputed MOFA factors and cluster memberships, not individual probes or metabolites

## Inputs

- MOFA factors from buccal DNA methylation data
- MOFA factors from urine metabolomics data
- SNF cluster memberships (cluster count determined by eigengap method in SNFpy)
- MCA behavioral dimension scores computed with FactoMineR
- Cohort sample identifiers including family structure for NTR twins

## Outputs

- association_statistics.csv with one row per tested predictor–behavioral-dimension pair per cohort, listing test type, test statistic, raw p-value, and FDR-corrected p-value

## How to apply

Load the MOFA factors derived from buccal DNA methylation and urine metabolomics, the SNF cluster memberships (cluster count chosen by the eigengap method in SNFpy), and the MCA behavioral dimension scores computed with FactoMineR. For the NTR twin cohort, fit Generalized Estimating Equation (GEE) models with the R package 'gee' using an exchangeable working correlation structure and Gaussian link to account for family relatedness, regressing each behavioral dimension score on each MOFA factor and on SNF cluster membership, running up to 100 iterations per model. For the LUMC-Curium cohort, compare behavioral dimension scores between SNF clusters using Mann–Whitney U tests with 2 clusters and Kruskal–Wallis tests with more than 2, and quantify omics–behavior relationships with Spearman's rank correlation. Correct all p-values for multiple testing across the factor–dimension pairs using FDR in both cohorts, then assemble a single association statistics table (cohort, predictor, behavioral dimension, test type, test statistic, raw and FDR-corrected p-values) written to association_statistics.csv.

## Related tools

- **gee (R package)** (Fits Generalized Estimating Equation models with exchangeable working correlation and Gaussian link to test associations in the NTR twin cohort while accounting for family relatedness)
- **SNFpy** (Provides the cluster memberships (optimal cluster number via eigengap method) tested as predictors of behavioral dimensions) — https://github.com/Xomics/Docker_containers.git
- **FactoMineR** (Supplies the MCA behavioral dimension scores used as response variables in the association models)
- **MOFA2** (Supplies the omics factors from buccal DNA methylation and urine metabolomics used as predictors) — https://github.com/Xomics/ACTIONdemonstrator_workflow.git
- **Nextflow** (Workflow manager that orchestrates the association-analysis module of the modular pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow.git
- **Docker** (Containers encapsulate the software dependencies (R packages, Python SNFpy) used by the association-testing processes) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --behavioral_data Synthetic_data/synthetic_cbcl_data.csv --phenotype_covariates Synthetic_data/synthetic_phenotype_covariates_data.csv --ids Synthetic_data/ACTIONdemonstrator_XOmics_IDS_synthetic.csv
```

## Evaluation signals

- association_statistics.csv contains one row per tested factor–dimension combination for both cohorts (NTR and LUMC-Curium)
- The correct test type is recorded per row: GEE with exchangeable correlation for NTR; Mann–Whitney U or Kruskal–Wallis for Curium depending on cluster count
- Every row includes an FDR-corrected p-value alongside the raw p-value
- Each row lists cohort, predictor (omics factor or cluster membership), behavioral dimension, test type, and test statistic
- GEE models in NTR converge within the up-to-100-iteration limit using the exchangeable working correlation structure

## Limitations

- The GEE approach with exchangeable correlation is specific to the related twin structure of the NTR cohort; it is not applied to the unrelated LUMC-Curium cohort, where non-parametric group tests and Spearman correlation are used instead
- The number of SNF clusters must first be chosen (eigengap method in SNFpy); a wrong cluster count changes whether Mann–Whitney U or Kruskal–Wallis is appropriate
- Privacy regulations prohibit sharing the real omics and clinical data; only synthetic data are provided, so results cannot be reproduced on the actual cohorts from the public repository

## Evidence

- [discussion] We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [discussion] Spearman rank correlation among omics and behavioral factors: "We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman’s rank correlation"
- [discussion] MOFA factors derived from buccal DNA methylation and urine metabolomics: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [discussion] Eigengap method for optimal cluster number in SNFpy: "uses the eigengap method [97] to find the optimal number of clusters"
- [readme] Nextflow pipeline with containers for software dependencies: "Nextflow automatically pulls all necessary Docker containers from Dockerhub, when using the default `nextflow.config` file."
