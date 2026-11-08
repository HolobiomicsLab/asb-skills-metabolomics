---
name: snf-cluster-group-comparison-testing
description: Use when integrated multi-omics clusters (e.g., SNF clusters from methylation/metabolomics)
  and behavioral dimension scores (e.g., MCA factors from CBCL data) must be statistically
  associated, accounting for family relatedness.
when_to_use_negative:
- Data are unrelated individuals with no family structure, so GEE with exchangeable
  correlation is unnecessary — ordinary regression or correlation suffices
- Cluster memberships have not yet been computed (SNF clustering with eigengap-based
  cluster count selection must run first)
- The task is multi-omics integration itself (MOFA factor derivation or SNF network
  construction) rather than downstream association testing
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R 'gee' package
  role: Fits GEE models with exchangeable working correlation and Gaussian link to
    test omics factor / SNF cluster associations with behavioral dimensions while
    adjusting for twin relatedness in NTR
- name: SNFpy
  role: Provides the SNF cluster memberships (cluster count via built-in eigengap
    function) that serve as the grouping predictor in the group-comparison tests
  repo: https://github.com/Xomics/Docker_containers.git
- name: FactoMineR
  role: Produces the MCA behavioral dimension scores that are the outcome variables
    in the association tests
- name: Nextflow
  role: Orchestrates the association-analysis module of the ACTIONdemonstrator workflow
    that runs the GEE/correlation tests and writes association_statistics.csv
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Containers pulled from Dockerhub provide the reproducible software environment
    for the association-testing processes
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:cf0428ae4c64280647220b5a525574df6d309a5762792ba6f6387390d6dc6dec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/Docker_containers.git
  tools:
  - Docker
  - FactoMineR
  - Nextflow
  - R 'gee' package
  - SNFpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# snf-cluster-group-comparison-testing

## Summary

Statistical association testing that links MOFA omics factors and SNF cluster memberships to MCA behavioral dimension scores, using GEE models for the related NTR twin cohort and non-parametric group comparisons plus Spearman correlations for the LUMC-Curium cohort, with FDR correction across all tested factor–dimension pairs.

## When to use

Use when integrated multi-omics clusters (e.g., SNF clusters from methylation/metabolomics) and behavioral dimension scores (e.g., MCA factors from CBCL data) must be statistically associated, accounting for family relatedness.

## When NOT to use

- Data are unrelated individuals with no family structure, so GEE with exchangeable correlation is unnecessary — ordinary regression or correlation suffices
- Cluster memberships have not yet been computed (SNF clustering with eigengap-based cluster count selection must run first)
- The task is multi-omics integration itself (MOFA factor derivation or SNF network construction) rather than downstream association testing

## Inputs

- MOFA factors derived from buccal DNA methylation and urine metabolomics
- SNF cluster memberships (optimal cluster count from SNFpy eigengap method)
- MCA behavioral dimension scores (FactoMineR, behavioral/CBCL data)
- Family/relatedness structure for the NTR twin cohort
- Cohort labels distinguishing NTR-ACTION from LUMC-Curium samples

## Outputs

- association_statistics.csv with one row per tested factor–dimension pair: cohort, predictor (omics factor or cluster membership), behavioral dimension, test type (GEE / Mann–Whitney U / Kruskal–Wallis / Spearman), test statistic, raw p-value, and FDR-corrected p-value

## How to apply

Load the MOFA factors from buccal DNA methylation and urine metabolomics, the SNF cluster memberships (cluster count chosen by the eigengap method in SNFpy), and the MCA behavioral dimension scores computed with FactoMineR. For the NTR cohort, fit GEE models with the R package 'gee' using an exchangeable working correlation structure and Gaussian link, regressing each behavioral dimension score on each omics factor and on SNF cluster membership, with up to 100 iterations per model to account for twin-family relatedness. For the LUMC-Curium cohort, compare behavioral dimension scores between SNF clusters with Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests (>2 clusters), and use Spearman's rank correlation among omics and behavioral factors. Correct all p-values for multiple testing across factor–dimension pairs with FDR in both cohorts, then assemble association_statistics.csv listing cohort, predictor, behavioral dimension, test type, test statistic, and raw and FDR-corrected p-values.

## Related tools

- **R 'gee' package** (Fits GEE models with exchangeable working correlation and Gaussian link to test omics factor / SNF cluster associations with behavioral dimensions while adjusting for twin relatedness in NTR)
- **SNFpy** (Provides the SNF cluster memberships (cluster count via built-in eigengap function) that serve as the grouping predictor in the group-comparison tests) — https://github.com/Xomics/Docker_containers.git
- **FactoMineR** (Produces the MCA behavioral dimension scores that are the outcome variables in the association tests)
- **Nextflow** (Orchestrates the association-analysis module of the ACTIONdemonstrator workflow that runs the GEE/correlation tests and writes association_statistics.csv) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Containers pulled from Dockerhub provide the reproducible software environment for the association-testing processes) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --behavioral_data Synthetic_data/synthetic_cbcl_data.csv --cbcl_labels Synthetic_data/cbcl_labels.csv
```

## Evaluation signals

- association_statistics.csv contains one row per tested factor–dimension combination for both cohorts (NTR and LUMC-Curium)
- The correct test type is recorded per row: GEE with exchangeable correlation for NTR; Mann–Whitney U (2 clusters) or Kruskal–Wallis (>2 clusters) for Curium
- Every row includes both a raw and an FDR-corrected p-value
- GEE models run to convergence within the 100-iteration limit for NTR models
- Spearman correlation results cover pairs among both omics and behavioral factors

## Limitations

- GEE with exchangeable correlation is specific to the related twin structure of NTR and cannot be applied as-is to unrelated cohorts
- Choice of test depends on the number of SNF clusters, which itself depends on the eigengap method; unstable cluster counts change the test type
- Personally identifiable omics/clinical data cannot be shared due to privacy regulations, so only a synthetic dataset is provided for validating the association module

## Evidence

- [discussion] Group comparison of behavioral dimension scores across SNF clusters: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [discussion] Spearman correlations among factors: "We determined correlations among the obtained factors capturing the omics and behavioral data, respectively, using Spearman’s rank correlation"
- [discussion] MCA applied to behavioral data with FactoMineR: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [readme] Nextflow invocation of the analysis workflow: "The typical command to run the workflow is:
``nnextflow run action.nf 
            --output dir/of/choice
            --mtblmcs_values Sythetic_data/synthetic_metabolomics.csv"
