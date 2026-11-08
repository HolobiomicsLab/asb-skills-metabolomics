---
name: eigengap-cluster-number-estimation
description: Use when you have a fused sample-similarity network (e.g., integrated
  DNA methylation and metabolomics similarity networks) and need a data-driven estimate
  of the optimal cluster count before spectral clustering.
when_to_use_negative:
- Input is raw omics feature tables that have not yet been standardized and converted
  into sample-by-sample similarity networks and fused via SNF
- The goal is supervised or covariate-adjusted sample grouping (e.g., GEE models)
  rather than unsupervised subgroup discovery
- The similarity network is not a valid fused affinity matrix (e.g., single un-fused
  data type where SNF fusion and spectral eigengap logic does not apply as designed)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SNFpy
  role: Python library whose built-in function applies the eigengap method to the
    fused network's spectral decomposition to find the optimal number of clusters;
    also provides spectral clustering for the k=2 and k=4 partitions
- name: Nextflow
  role: Workflow manager that packages the SNF clustering and cluster-number estimation
    steps as modular pipeline processes
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
- name: Docker
  role: Container format pulled by Nextflow to pin the SNFpy runtime environment
  repo: https://github.com/Xomics/Docker_containers.git
- name: Apptainer/Singularity
  role: Container format (.sif) used to execute the SNF clustering steps in constrained
    environments such as the Digital Research Environment
  repo: https://github.com/Xomics/Docker_containers.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:5fa3e19f84ef01a096a15750b5b365942c06567ed7ca63e34e5b7702c4c40a4b
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# eigengap-cluster-number-estimation

## Summary

Estimates the optimal number of clusters for a fused SNF similarity network by applying the eigengap method to the network's spectral decomposition, as implemented in SNFpy's built-in function within the ACTIONdemonstrator multi-omics workflow.

## When to use

Use when you have a fused sample-similarity network (e.g., integrated DNA methylation and metabolomics similarity networks) and need a data-driven estimate of the optimal cluster count before spectral clustering.

## When NOT to use

- Input is raw omics feature tables that have not yet been standardized and converted into sample-by-sample similarity networks and fused via SNF
- The goal is supervised or covariate-adjusted sample grouping (e.g., GEE models) rather than unsupervised subgroup discovery
- The similarity network is not a valid fused affinity matrix (e.g., single un-fused data type where SNF fusion and spectral eigengap logic does not apply as designed)

## Inputs

- Fused/integrated SNF sample-similarity network per cohort (derived from standardized methylation and metabolomics distance/similarity matrices)
- Sample identifiers aligned per cohort (NTR and LUMC-Curium)

## Outputs

- Estimated optimal number of clusters per cohort (eigengap-based)
- Cluster-assignment table (snf_cluster_assignments.csv) with columns for sample ID, cohort, k, and cluster label for k=2 and k=4 solutions

## How to apply

First fuse per-data-type similarity networks (buccal DNA methylation and urine metabolomics, standardized per data type) into an integrated similarity network with the SNF algorithm. Then run SNFpy's built-in cluster-number estimation, which applies the eigengap method to the spectral decomposition of the fused network to identify the k with the largest eigengap. Use the estimated k to partition samples with SNFpy spectral clustering, generating both 2-cluster (k=2) and 4-cluster (k=4) solutions as done for the NTR and LUMC-Curium cohorts. Retain sample identifiers, cohort, k, and cluster labels so downstream group comparisons (e.g., Mann–Whitney U / Kruskal–Wallis tests of MCA behavioral scores) can be joined on sample ID.

## Related tools

- **SNFpy** (Python library whose built-in function applies the eigengap method to the fused network's spectral decomposition to find the optimal number of clusters; also provides spectral clustering for the k=2 and k=4 partitions)
- **Nextflow** (Workflow manager that packages the SNF clustering and cluster-number estimation steps as modular pipeline processes) — https://github.com/Xomics/ACTIONdemonstrator_workflow
- **Docker** (Container format pulled by Nextflow to pin the SNFpy runtime environment) — https://github.com/Xomics/Docker_containers.git
- **Apptainer/Singularity** (Container format (.sif) used to execute the SNF clustering steps in constrained environments such as the Digital Research Environment) — https://github.com/Xomics/Docker_containers.git

## Examples

```
from snf import compute; best_ks, second_ks = compute.get_n_clusters(fused_network)
```

## Evaluation signals

- Each input sample receives exactly one cluster label for every k and cohort combination in snf_cluster_assignments.csv
- The k=2 solution yields exactly 2 distinct clusters and the k=4 solution yields exactly 4 distinct clusters per cohort
- The estimated optimal k from the eigengap method is consistent with the range of k values explored (clusters at k=2 and k=4 were both defensible in NTR and LUMC-Curium)
- The fused network fed to the eigengap estimation is a single square sample-by-sample similarity matrix per cohort with all samples present after methylation/metabolomics alignment

## Limitations

- Privacy regulations prohibit sharing the real identifiable omics data, so only synthetic data (Synthetic_data/) is provided; estimated k on synthetic data will not reproduce the real cohort results
- The eigengap heuristic depends on the spectral decomposition of the fused network, so its answer is sensitive to upstream normalization and which probes/features entered the similarity computation (e.g., top 10% most variable methylation probes)
- Describing unsupervised learning workflow steps like this one in RO-Crate required the eNanoMapper Ontology because no matching EDAM term was available

## Evidence

- [discussion] SNF integrated similarity network construction and clustering: "we constructed integrated similarity networks with SNF [72]. The optimal numbers of clusters were determined using a built-in function of the Python library SNFpy"
- [discussion] Eigengap method for optimal cluster number: "uses the eigengap method [97] to find the optimal number of clusters"
- [discussion] Integrated networks defined at 2 and 4 clusters: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic example data: "Synthetic data is provided as example data in `Synthetic_data/`. This directory contains the necessary (omics) data files to run the workflow"
