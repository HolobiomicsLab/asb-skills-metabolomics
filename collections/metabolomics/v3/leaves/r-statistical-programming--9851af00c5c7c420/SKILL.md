---
name: r-statistical-programming
description: Use when categorical behavioral item data (e.g., parent-rated CBCL externalizing
  items) or paired omics matrices need unsupervised factor extraction and variance
  reporting inside a Nextflow-pipeline R environment.
when_to_use_negative:
- The data are continuous omics measurements (methylation betas, metabolite intensities)
  rather than categorical behavioral items — MOFA2, not MCA, is the paper's method
  for those.
- The raw cohort data have not yet been converted to standardized formats — run the
  Xomics/ACTIONdemonstrator_preprocessing workflow first.
edam_operation: http://edamontology.org/operation_2939
tools:
- name: FactoMineR
  role: R library used to run MCA on the categorical CBCL behavioral items to obtain
    behavioral factors
- name: MOFA2
  role: R/Bioconductor library used for factor extraction from buccal DNA methylation
    and urine metabolomics data
- name: Nextflow
  role: Workflow manager that orchestrates the R analysis modules and containerized
    dependencies
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker/Singularity
  role: Containers (e.g., r-base-phenotypes) pinning the R runtime and libraries for
    the workflow processes
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:04f53cd589024606d8088b8ce07c57f048ab4923eb97ee755aaea641cdfa820b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Docker/Singularity
  - FactoMineR
  - MOFA2
  - Nextflow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# r-statistical-programming

## Summary

R statistical programming underpins the factor-analysis and group-comparison steps of the ACTION multi-omics workflow: MCA of categorical ASEBA CBCL behavior items with FactoMineR (v2.4), MOFA2 factor extraction from methylation/metabolomics data, and Mann–Whitney U, Kruskal–Wallis, GEE, and Spearman correlation testing.

## When to use

Use when categorical behavioral item data (e.g., parent-rated CBCL externalizing items) or paired omics matrices need unsupervised factor extraction and variance reporting inside a Nextflow-pipeline R environment.

## When NOT to use

- The data are continuous omics measurements (methylation betas, metabolite intensities) rather than categorical behavioral items — MOFA2, not MCA, is the paper's method for those.
- The raw cohort data have not yet been converted to standardized formats — run the Xomics/ACTIONdemonstrator_preprocessing workflow first.

## Inputs

- Imputed parent-rated ASEBA CBCL externalizing behavior item scores (categorical, CSV; e.g., Synthetic_data/synthetic_cbcl_data.csv)
- CBCL item label mapping (Synthetic_data/cbcl_labels.csv)
- Buccal DNA methylation and urine metabolomics intensity matrices (for MOFA2 factor extraction)

## Outputs

- MCA dimension variance table (e.g., mca_dimension_variance.csv with dimension index and fraction of variation explained)
- Behavioral factor/dimension scores used in downstream SNF-cluster group comparisons and Spearman correlations

## How to apply

Load the imputed parent-rated externalizing behavior item scores (e.g., Synthetic_data/synthetic_cbcl_data.csv) into R and run multiple correspondence analysis (MCA) with the R library FactoMineR (v2.4), as done in the ACTIONdemonstrator workflow to obtain factors for the behavioral data. Extract the MCA eigenvalues and compute the fraction of total variation explained by each dimension, reporting them as a table with columns for dimension index and fraction of variation explained (NTR dimensions 1–3 explain ~16%/9%/<5%; LUMC-Curium's first 2 dimensions together ~30%). For continuous omics data, use the R/Bioconductor library MOFA2 (1.3.4) instead to obtain factors for buccal DNA methylation and urine metabolomics. R steps run as modules of the Nextflow pipeline using Docker/Singularity containers (e.g., r-base-phenotypes) built from the Xomics/Docker_containers repository, and results such as the per-dimension variance table are written to files like mca_dimension_variance.csv.

## Related tools

- **FactoMineR** (R library used to run MCA on the categorical CBCL behavioral items to obtain behavioral factors)
- **MOFA2** (R/Bioconductor library used for factor extraction from buccal DNA methylation and urine metabolomics data)
- **Nextflow** (Workflow manager that orchestrates the R analysis modules and containerized dependencies) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker/Singularity** (Containers (e.g., r-base-phenotypes) pinning the R runtime and libraries for the workflow processes) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --behavioral_data Synthetic_data/synthetic_cbcl_data.csv --cbcl_labels Synthetic_data/cbcl_labels.csv --ids Synthetic_data/ACTIONdemonstrator_XOmics_IDS_synthetic.csv
```

## Evaluation signals

- Computed per-dimension variance fractions reproduce the reported figures: NTR dimension 1 ~16%, dimension 2 ~9%, dimension 3 <5%; LUMC-Curium first 2 dimensions ~30% combined.
- Output table mca_dimension_variance.csv contains the required columns: dimension index and fraction of variation explained.
- MCA is run on categorical behavioral items with FactoMineR version 2.4, matching the published workflow version.
- Behavioral dimension scores feed the downstream group comparisons (Mann–Whitney U / Kruskal–Wallis / GEE) against SNF clusters without format errors.
- R steps execute inside the pinned containers pulled via nextflow.config, so the environment is reproducible.

## Limitations

- Only synthetic data (Synthetic_data/) can be shared because privacy regulations prohibit sharing personally identifiable omics or clinical data, so variance fractions on synthetic data may differ from cohort-specific reference ranges.
- Per-dimension variance explained varies by cohort (NTR vs. LUMC-Curium); reference ranges are cohort-specific and should not be applied interchangeably.
- A specific FactoMineR version (2.4) is required; other versions may change results or behavior.

## Evidence

- [discussion] MCA applied to behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [discussion] MOFA2 factor extraction in R: "We applied MOFA using the R/Bioconductor library MOFA2 (1.3.4) [67, 95] to obtain factors for the buccal DNA methylation and urine metabolomics data"
- [abstract] Containerized Nextflow pipeline: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
- [readme] Synthetic example data in repository: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
