---
name: multi-omics-network-fusion
description: Use when two or more standardized sample-by-feature omics tables (e.g.
  buccal DNA methylation betas and urine metabolomics intensities) must be integrated
  into a fused sample similarity network to identify subject subgroups per cohort.
when_to_use_negative:
- When the goal is factor-based integration rather than subgroup clustering — the
  same workflow uses MOFA2 for the omics factors and FactoMineR MCA for behavioral
  data instead of SNF
- When inputs are raw instrument files or unstandardized phenotypes — the workflow
  requires data already converted into standardized file formats by the ACTIONdemonstrator_preprocessing
  workflow
- When analyzing a single omics data type — SNF requires multiple data types to fuse
  similarity networks
edam_operation: http://edamontology.org/operation_3432
tools:
- name: SNFpy
  role: Python library providing similarity-network normalization, the SNF fusion
    algorithm, eigengap-based optimal cluster number estimation, and spectral clustering
- name: Nextflow
  role: Workflow manager orchestrating the SNF clustering module within the action.nf
    pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime for the SNF/Python software environment, pulled automatically
    via nextflow.config
  repo: https://github.com/Xomics/Docker_containers.git
- name: Apptainer/Singularity
  role: Container format (.sif) used to run the SNF module in restricted environments
    such as the DRE via dre.config
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:38417e23d7b035539f3594eefdc2eff966710686b8f57a4a4ecd840c390862b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Apptainer/Singularity
  - Docker
  - Nextflow
  - SNFpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multi-omics-network-fusion

## Summary

Fuses DNA methylation and metabolomics similarity networks into an integrated sample similarity network with Similarity Network Fusion (SNF), then partitions individuals into 2- and 4-cluster solutions. This is the ACTIONdemonstrator workflow's subgroup-discovery step, implemented in Python's SNFpy inside a Nextflow-managed containerized pipeline.

## When to use

Use when two or more standardized sample-by-feature omics tables (e.g. buccal DNA methylation betas and urine metabolomics intensities) must be integrated into a fused sample similarity network to identify subject subgroups per cohort.

## When NOT to use

- When the goal is factor-based integration rather than subgroup clustering — the same workflow uses MOFA2 for the omics factors and FactoMineR MCA for behavioral data instead of SNF
- When inputs are raw instrument files or unstandardized phenotypes — the workflow requires data already converted into standardized file formats by the ACTIONdemonstrator_preprocessing workflow
- When analyzing a single omics data type — SNF requires multiple data types to fuse similarity networks

## Inputs

- Standardized DNA methylation beta-value table (synthetic_epigenomics.csv, sample-by-probe CSV)
- Standardized urine metabolomics intensity table (synthetic_metabolomics.csv, sample-by-metabolite CSV)
- Epigenomics metadata table (synthetic_epigenomics_meta.csv) for sample/cohort alignment
- X-omics sample ID table (ACTIONdemonstrator_XOmics_IDS_synthetic.csv)

## Outputs

- Fused integrated sample similarity network per cohort (methylation + metabolomics)
- Cluster-assignment table snf_cluster_assignments.csv with columns: sample ID, cohort, k (2 and 4), cluster label
- Estimated optimal number of clusters per cohort via the eigengap method

## How to apply

Load the standardized methylation and metabolomics tables (e.g. Synthetic_data/synthetic_epigenomics.csv and synthetic_metabolomics.csv) plus epigenomics metadata with pandas and align samples per cohort (NTR and LUMC-Curium). Standardize features per data type and compute sample-by-sample distance matrices, converting them into normalized similarity networks with SNFpy's normalization utilities. Fuse the methylation and metabolomics similarity networks into an integrated network with the SNF algorithm, then estimate the optimal number of clusters with SNFpy's built-in eigengap method applied to the network's spectral decomposition. Partition samples with SNFpy spectral clustering for both k=2 and k=4 solutions and write per-cohort, per-k assignments to snf_cluster_assignments.csv with columns for sample ID, cohort, k, and cluster label.

## Related tools

- **SNFpy** (Python library providing similarity-network normalization, the SNF fusion algorithm, eigengap-based optimal cluster number estimation, and spectral clustering)
- **Nextflow** (Workflow manager orchestrating the SNF clustering module within the action.nf pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime for the SNF/Python software environment, pulled automatically via nextflow.config) — https://github.com/Xomics/Docker_containers.git
- **Apptainer/Singularity** (Container format (.sif) used to run the SNF module in restricted environments such as the DRE via dre.config) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv --ids Synthetic_data/ACTIONdemonstrator_XOmics_IDS_synthetic.csv
```

## Evaluation signals

- snf_cluster_assignments.csv exists and contains a row for every input sample in each cohort
- Every sample receives exactly one cluster label per k and per cohort — no missing or duplicate assignments
- The k=2 solution yields exactly 2 distinct clusters and the k=4 solution exactly 4 distinct clusters
- The eigengap method applied to the fused network's spectral decomposition produces a defined optimal k estimate per cohort
- Cluster labels align to sample identifiers across the methylation and metabolomics inputs (same sample set fused)

## Limitations

- Privacy regulations prohibit sharing the real, potentially personally identifiable omics and clinical data; only a synthetic dataset is distributed, so results on shared data are demonstrative rather than scientific findings
- Cluster number determination relies on the eigengap heuristic; the 2- and 4-cluster solutions were applied per cohort (NTR and LUMC-Curium) and downstream testing (Mann-Whitney U for k=2, Kruskal-Wallis/GEE) accompanies rather than validates the clustering itself
- Describing unsupervised learning steps with standard metadata terms is limited: no matching EDAM term existed, so eNanoMapper Ontology terms had to be used in the RO-Crate

## Evidence

- [discussion] SNF network construction and clustering in the published workflow: "we constructed integrated similarity networks with SNF [72]. The optimal numbers of clusters were determined using a built-in function of the Python library SNFpy"
- [discussion] Optimal cluster number estimation method: "uses the eigengap method [97] to find the optimal number of clusters"
- [discussion] Downstream statistical testing of the SNF clusters: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [readme] Workflow expects standardized inputs: "This workflows ingests molecular -omics profiling and phenotypic data that is already converted into standardized file formats."
- [readme] Synthetic example data provided to run the workflow: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow + the ISA-Tab files describing the experimental metadata."
