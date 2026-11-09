---
name: sample-similarity-network-construction
description: Use when multi-omics feature tables for the same samples (e.g., methylation
  betas and metabolite intensities) must be integrated into a single sample-by-sample
  similarity network to identify subject subgroups via clustering.
when_to_use_negative:
- The goal is latent-factor extraction (e.g., MOFA2 factors) rather than discrete
  sample subgrouping via network fusion
- The data types describe different sample sets that cannot be aligned sample-by-sample
  per cohort
- Raw data are not yet standardized/preprocessed (standardization and QC are handled
  upstream in the ACTIONdemonstrator preprocessing workflow)
edam_operation: http://edamontology.org/operation_3935
edam_topics: []
tools:
- name: SNFpy
  role: Python library used to normalize similarity matrices, run the SNF fusion algorithm,
    estimate the optimal number of clusters via the eigengap method, and perform spectral
    clustering (k=2 and k=4)
- name: Nextflow
  role: Workflow manager that orchestrates the SNF clustering step within the modular
    ACTIONdemonstrator pipeline (action.nf)
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Containerizes the software dependencies for the pipeline steps (e.g., miniconda-snf
    image), pulled automatically from Dockerhub via nextflow.config
  repo: https://github.com/Xomics/Docker_containers.git
- name: Apptainer/Singularity
  role: Singularity (.sif) containers allocated to Nextflow processes (dre.config)
    for secure execution environments such as the DRE
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:a2de99ec0a9c29867acefa1201a5944e04ee2205c28e098bf601578b69c12da3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - Apptainer/Singularity
  - Docker
  - Nextflow
  - SNFpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# sample-similarity-network-construction

## Summary

Fuses standardized per-data-type sample similarity networks (DNA methylation and urine metabolomics) into one integrated similarity network with Similarity Network Fusion (SNF) and partitions samples into k=2 and k=4 clusters, as implemented in the ACTIONdemonstrator workflow for the NTR and LUMC-Curium cohorts.

## When to use

Use when multi-omics feature tables for the same samples (e.g., methylation betas and metabolite intensities) must be integrated into a single sample-by-sample similarity network to identify subject subgroups via clustering.

## When NOT to use

- The goal is latent-factor extraction (e.g., MOFA2 factors) rather than discrete sample subgrouping via network fusion
- The data types describe different sample sets that cannot be aligned sample-by-sample per cohort
- Raw data are not yet standardized/preprocessed (standardization and QC are handled upstream in the ACTIONdemonstrator preprocessing workflow)

## Inputs

- DNA methylation feature table (e.g., synthetic_epigenomics.csv)
- Metabolomics feature table (e.g., synthetic_metabolomics.csv)
- Epigenomics metadata CSV for sample alignment (e.g., synthetic_epigenomics_meta.csv)

## Outputs

- Fused integrated sample similarity network per cohort
- snf_cluster_assignments.csv with sample ID, cohort, k (2 and 4), and cluster label columns

## How to apply

Load the per-cohort omics tables (e.g., Synthetic_data/synthetic_epigenomics.csv and synthetic_metabolomics.csv) plus epigenomics metadata with pandas and align samples. Standardize features per data type, then compute sample-by-sample distance matrices and convert them into normalized similarity networks using SNFpy's normalization utilities. Fuse the methylation and metabolomics networks into one integrated network with SNF, and estimate the optimal cluster number per cohort using SNFpy's built-in eigengap method on the fused network's spectral decomposition. Partition samples with SNFpy spectral clustering for k=2 and k=4, retaining sample identifiers and cluster labels. Write per-cohort, per-k assignments to snf_cluster_assignments.csv with columns for sample ID, cohort, k, and cluster label.

## Related tools

- **SNFpy** (Python library used to normalize similarity matrices, run the SNF fusion algorithm, estimate the optimal number of clusters via the eigengap method, and perform spectral clustering (k=2 and k=4))
- **Nextflow** (Workflow manager that orchestrates the SNF clustering step within the modular ACTIONdemonstrator pipeline (action.nf)) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Containerizes the software dependencies for the pipeline steps (e.g., miniconda-snf image), pulled automatically from Dockerhub via nextflow.config) — https://github.com/Xomics/Docker_containers.git
- **Apptainer/Singularity** (Singularity (.sif) containers allocated to Nextflow processes (dre.config) for secure execution environments such as the DRE) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --mtblmcs_values Synthetic_data/synthetic_metabolomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv
```

## Evaluation signals

- Every input sample in each cohort receives exactly one cluster label for each k (2 and 4)
- The k=2 solution yields exactly 2 distinct clusters and the k=4 solution exactly 4 distinct clusters
- snf_cluster_assignments.csv exists and contains the expected columns: sample ID, cohort, k, cluster label
- Optimal cluster number estimates are reproducible via the eigengap method applied to each cohort's fused network spectral decomposition
- Cluster assignment tables are produced for both NTR and LUMC-Curium cohorts

## Limitations

- Privacy regulations prohibit sharing the real, potentially personally identifiable omics and clinical data; only a synthetic dataset (Synthetic_data/) is distributed, so results must be reproduced against synthetic inputs or authorized access to ACTION Biomarker Study data
- Only k=2 and k=4 solutions were defined in the study for both NTR and LUMC-Curium; other k values require re-running the eigengap/spectral clustering step
- The workflow requires SNFpy, Nextflow, and containers (Docker/Singularity), and use of workflow management systems may require initial training of researchers

## Evidence

- [discussion] SNF integrated similarity network construction and clustering: "we constructed integrated similarity networks with SNF [72]. The optimal numbers of clusters were determined using a built-in function of the Python library SNFpy"
- [discussion] Eigengap method for optimal cluster number: "uses the eigengap method [97] to find the optimal number of clusters"
- [abstract] Modular Nextflow pipeline: "The analysis workflow was implemented as a modular pipeline in the workflow manager Nextflow, including containers with software dependencies."
- [readme] Synthetic example data: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
- [readme] Container provisioning: "Nextflow automatically pulls all necessary Docker containers from Dockerhub, when using the default `nextflow.config` file."
