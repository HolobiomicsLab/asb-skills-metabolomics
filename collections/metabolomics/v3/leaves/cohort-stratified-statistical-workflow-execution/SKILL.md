---
name: cohort-stratified-statistical-workflow-execution
description: Use when testing whether MOFA factors (buccal DNA methylation, urine
  metabolomics) or SNF cluster memberships are associated with MCA behavioral dimension
  scores in cohorts with differing relatedness structure (twin cohort vs. unrelated
  children).
when_to_use_negative:
- Inputs are still raw omics measurements (e.g., unfiltered Infinium MethylationEPIC
  betas or unnormalized urine metabolite intensities) rather than reduced MOFA factors
  and SNF clusters — run the upstream factor/clustering modules first.
- The analysis goal is metabolomics/methylation QC, sample filtering, or MOFA/SNF
  model fitting rather than testing associations with behavioral dimensions.
- The cohort has no grouping structure and the goal is simple exploratory correlation
  only — a standalone Spearman analysis may suffice instead of the stratified GEE/rank-test
  design.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R package 'gee'
  role: Fits GEE models with exchangeable working correlation and Gaussian link to
    regress behavioral dimension scores on omics factors and SNF cluster membership
    while accounting for NTR family relatedness
- name: Nextflow
  role: Workflow manager that orchestrates the association-testing module as part
    of the modular ACTIONdemonstrator pipeline (action.nf)
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Containers pulled from Dockerhub (or built from Xomics/Docker_containers)
    provide the R/Python runtime dependencies for the association statistics processes
  repo: https://github.com/Xomics/Docker_containers.git
- name: SNFpy
  role: Supplies the cluster memberships (eigengap-selected k) that serve as predictors
    in the cluster vs. behavioral-dimension comparisons
- name: FactoMineR
  role: Produces the MCA behavioral dimension scores that are the dependent variables
    in the association tests
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:6f529549bdd721adefaeb96703bf63b17521bb31202ead6176119cc5ef796ecd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker
  - FactoMineR
  - Nextflow
  - R package 'gee'
  - SNFpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cohort-stratified-statistical-workflow-execution

## Summary

Cohort-stratified association testing module of the ACTIONdemonstrator Nextflow workflow: it regresses MCA behavioral dimension scores on MOFA omics factors and SNF cluster memberships, using GEE (R 'gee' package, exchangeable correlation, Gaussian link) to absorb twin relatedness in NTR and non-parametric tests plus Spearman correlation in LUMC-Curium, with FDR correction across all tested pairs.

## When to use

Use when testing whether MOFA factors (buccal DNA methylation, urine metabolomics) or SNF cluster memberships are associated with MCA behavioral dimension scores in cohorts with differing relatedness structure (twin cohort vs. unrelated children).

## When NOT to use

- Inputs are still raw omics measurements (e.g., unfiltered Infinium MethylationEPIC betas or unnormalized urine metabolite intensities) rather than reduced MOFA factors and SNF clusters — run the upstream factor/clustering modules first.
- The analysis goal is metabolomics/methylation QC, sample filtering, or MOFA/SNF model fitting rather than testing associations with behavioral dimensions.
- The cohort has no grouping structure and the goal is simple exploratory correlation only — a standalone Spearman analysis may suffice instead of the stratified GEE/rank-test design.

## Inputs

- MOFA factor scores from buccal DNA methylation and urine metabolomics data
- SNF cluster memberships per subject (cluster count from the SNFpy eigengap method)
- MCA behavioral dimension scores computed with FactoMineR (FactoMineR 2.4)
- Twin/family relatedness structure of the NTR cohort (required by GEE with exchangeable correlation)

## Outputs

- association_statistics.csv with one row per tested factor–dimension pair: cohort, predictor, behavioral dimension, test type, test statistic, raw p-value, FDR-corrected p-value

## How to apply

Load the MOFA2-derived omics factor scores, the SNF cluster memberships (k chosen by the eigengap method in SNFpy), and the FactoMineR MCA behavioral dimension scores. For the NTR twin cohort, fit one Generalized Estimating Equation per factor–dimension pair with the R package 'gee', specifying an exchangeable working correlation structure and Gaussian link so family relatedness is absorbed; cap each model at 100 iterations. For the LUMC-Curium cohort of unrelated children, compare dimension scores between SNF clusters with Mann–Whitney U tests when k=2 and Kruskal–Wallis tests when k>2, and quantify factor–factor relationships with Spearman's rank correlation. Correct every raw p-value for multiple testing across all tested factor–dimension pairs with FDR in both cohorts. Assemble a single table recording cohort, predictor (omics factor or cluster membership), behavioral dimension, test type, test statistic, and raw plus FDR-corrected p-values, and write it to association_statistics.csv.

## Related tools

- **R package 'gee'** (Fits GEE models with exchangeable working correlation and Gaussian link to regress behavioral dimension scores on omics factors and SNF cluster membership while accounting for NTR family relatedness)
- **Nextflow** (Workflow manager that orchestrates the association-testing module as part of the modular ACTIONdemonstrator pipeline (action.nf)) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Containers pulled from Dockerhub (or built from Xomics/Docker_containers) provide the R/Python runtime dependencies for the association statistics processes) — https://github.com/Xomics/Docker_containers.git
- **SNFpy** (Supplies the cluster memberships (eigengap-selected k) that serve as predictors in the cluster vs. behavioral-dimension comparisons)
- **FactoMineR** (Produces the MCA behavioral dimension scores that are the dependent variables in the association tests)

## Examples

```
nextflow run action.nf --output dir/of/choice --behavioral_data Synthetic_data/synthetic_cbcl_data.csv
```

## Evaluation signals

- association_statistics.csv contains exactly one row per tested factor–dimension combination for both cohorts (NTR and LUMC-Curium)
- Each row records the correct test type: GEE with exchangeable correlation for NTR; Mann–Whitney U (2 clusters) or Kruskal–Wallis (>2 clusters) plus Spearman correlation for Curium
- Every row carries both a raw and an FDR-corrected p-value
- NTR GEE models converged within the 100-iteration cap for each factor–dimension pair
- FDR correction was applied jointly across all tested factor–dimension pairs within each cohort

## Limitations

- GEE models are capped at 100 iterations per model; non-convergent factor–dimension pairs will fail or need iteration-limit adjustments
- Test choice is fixed by cluster count in Curium (Mann–Whitney U for 2 clusters, Kruskal–Wallis for >2), so changing k in SNFpy changes the statistical test applied
- Privacy regulations prohibit sharing the real NTR/LUMC-Curium omics and clinical data, so only a synthetic dataset in Synthetic_data/ can be used to demonstrate/re-run this module

## Evidence

- [discussion] We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [discussion] Spearman correlations among factors: [evidence span withheld — anchor status: not_found]
- [discussion] MOFA factor inputs: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [readme] Standardized input requirement: "This workflows ingests molecular -omics profiling and phenotypic data that is already converted into standardized file formats."
