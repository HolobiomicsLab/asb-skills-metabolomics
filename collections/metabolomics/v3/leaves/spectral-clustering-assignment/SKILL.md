---
name: spectral-clustering-assignment
description: Use when a fused sample-by-sample similarity network (e.g., from SNF
  over methylation and metabolomics distance matrices) must be partitioned into subgroups,
  with cluster labels retained per sample and cohort.
when_to_use_negative:
- Input is raw omics feature tables rather than a fused sample similarity network
  — feature standardization, distance computation, and SNF fusion must happen first
- The analysis goal is supervised prediction of an external phenotype (e.g., behavioral
  scores) rather than unsupervised subgroup discovery from the integrated omics networks
- Only a single data type is available; SNF fusion requires at least two similarity
  networks (here methylation and metabolomics)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: SNFpy
  role: Provides SNF fusion, the eigengap-based estimation of optimal cluster number,
    and spectral clustering of the fused network for k=2 and k=4
- name: Nextflow
  role: Workflow manager that packages and executes this clustering step as a modular
    process within the ACTIONdemonstrator pipeline (action.nf)
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container runtime for the clustering process; containers are pulled automatically
    via nextflow.config
  repo: https://github.com/Xomics/Docker_containers.git
- name: Apptainer/Singularity
  role: Alternative container runtime; .sif files are allocated to Nextflow processes
    via the dre.config file
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:702a82c235ad79175dea4c1b2cd4ee20220ceeae9cbd9b2c5ad1c9834acd2187
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
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spectral-clustering-assignment

## Summary

Partitions samples of an integrated SNF similarity network into k=2 and k=4 subgroups using SNFpy's spectral clustering, after estimating the optimal cluster number with the eigengap method. This is the subgroup-identification step of the ACTIONdemonstrator multi-omics workflow applied to buccal DNA methylation and urine metabolomics data from the NTR and LUMC-Curium cohorts.

## When to use

Use when a fused sample-by-sample similarity network (e.g., from SNF over methylation and metabolomics distance matrices) must be partitioned into subgroups, with cluster labels retained per sample and cohort.

## When NOT to use

- Input is raw omics feature tables rather than a fused sample similarity network — feature standardization, distance computation, and SNF fusion must happen first
- The analysis goal is supervised prediction of an external phenotype (e.g., behavioral scores) rather than unsupervised subgroup discovery from the integrated omics networks
- Only a single data type is available; SNF fusion requires at least two similarity networks (here methylation and metabolomics)

## Inputs

- Fused integrated sample similarity network produced by SNF from methylation and metabolomics similarity matrices
- Cohort-aligned sample identifiers from synthetic_epigenomics.csv and synthetic_metabolomics.csv
- Cohort labels (NTR, LUMC-Curium) and epigenomics metadata
- Target cluster counts k=2 and k=4

## Outputs

- snf_cluster_assignments.csv with columns: sample ID, cohort, k, cluster label
- Per-cohort, per-k cluster label tables ensuring every input sample receives exactly one label per k

## How to apply

Load the cohort-aligned DNA methylation (synthetic_epigenomics.csv) and metabolomics (synthetic_metabolomics.csv) tables, standardize features per data type, and compute sample-by-sample distance matrices converted to normalized similarity networks with SNFpy's normalization utilities. Fuse the networks into an integrated similarity network with the SNF algorithm as implemented in SNFpy. Estimate the optimal number of clusters using SNFpy's built-in function, which applies the eigengap method to the fused network's spectral decomposition. Partition samples with SNFpy spectral clustering for both the 2-cluster (k=2) and 4-cluster (k=4) solutions, keeping sample identifiers and cohort labels alongside cluster labels. Write per-cohort, per-k cluster assignments to snf_cluster_assignments.csv with columns for sample ID, cohort, k, and cluster label, so downstream group comparisons (e.g., Mann-Whitney U for 2 clusters, Kruskal-Wallis for 4) can be run on behavioral dimension scores.

## Related tools

- **SNFpy** (Provides SNF fusion, the eigengap-based estimation of optimal cluster number, and spectral clustering of the fused network for k=2 and k=4)
- **Nextflow** (Workflow manager that packages and executes this clustering step as a modular process within the ACTIONdemonstrator pipeline (action.nf)) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container runtime for the clustering process; containers are pulled automatically via nextflow.config) — https://github.com/Xomics/Docker_containers.git
- **Apptainer/Singularity** (Alternative container runtime; .sif files are allocated to Nextflow processes via the dre.config file) — https://github.com/Xomics/Docker_containers.git

## Examples

```
nextflow run action.nf --output dir/of/choice --epigenomics_values Synthetic_data/synthetic_epigenomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv
```

## Evaluation signals

- Each output CSV exists and every input sample receives exactly one cluster label per k and per cohort
- The 2-cluster solution yields exactly 2 distinct clusters and the 4-cluster solution exactly 4 distinct clusters
- Cluster-assignment table schema matches: sample ID, cohort, k, cluster label
- The eigengap estimate from SNFpy's built-in function is reported and consistent with the k values selected (2 and 4)
- All samples from both cohorts (NTR and LUMC-Curium) appear in the assignments with no duplicates or missing IDs

## Limitations

- Only the synthetic dataset is shared because privacy regulations prohibit sharing potentially personally identifiable omics measurements or clinical information, so reproducibility is demonstrated on synthetic data
- k=2 and k=4 were fixed as the evaluated solutions; deviations from the eigengap-optimal k in other datasets may require re-running the optimal-cluster-number estimation
- To describe workflow steps with unsupervised learning in RO-Crate, the eNanoMapper Ontology had to be used because no matching term was available in the recommended EDAM ontology

## Evidence

- [discussion] SNF integrated similarity network construction and clustering: "we constructed integrated similarity networks with SNF [72]. The optimal numbers of clusters were determined using a built-in function of the Python library SNFpy"
- [discussion] Eigengap method for optimal cluster number: "uses the eigengap method [97] to find the optimal number of clusters"
- [discussion] Group comparison of behavioral dimension scores across SNF clusters: "We tested whether the behavioral dimension scores from MCA differ between children in the different SNF clusters, using Mann–Whitney U tests (2 clusters) or Kruskal–Wallis tests"
- [readme] Running the workflow on synthetic data: "--epigenomics_values Synthetic_data/synthetic_epigenomics.csv --epigenomics_meta Synthetic_data/synthetic_epigenomics_meta.csv"
