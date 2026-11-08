---
name: cluster-agreement-statistics-reproduction
description: Use when you have MS-Cluster clusterinfo.tsv clusters for a ProteomeXchange
  dataset plus a parallel DB-Search purity estimate, and you need to quantify how
  well MS-RT purity matches a database-search ground truth at the per-cluster level.
when_to_use_negative:
- The dataset is metabolomics-only with no parallel database search identifications
  — in that case MS-RT must be applied directly as the evaluation method rather than
  validated against DB-Search, since 'in mass spectrometry-based metabolomics, FDR
  controlled identifications are not currently practical'.
- The clustering tool output lacks the required '#RetTime' column (e.g. Falcon or
  MaRaCluster output that has not been merged with retention time), so MS-RT purity
  cannot be computed as-is.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS-Cluster
  role: Produces the clustered spectra (clusterinfo.tsv) whose per-cluster purity
    is evaluated by both MS-RT and DB-Search
  repo: https://gnps2.org/workflowinput?workflowname=classical_networking_workflow
- name: MS-RT
  role: Computes per-cluster purity from MS1 retention time consistency within clusters
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: DB-Search
  role: Provides the reference per-cluster purity from MS-GF+ FDR-controlled identifications
    for validation
- name: msconvert
  role: Converts vendor raw files to mzML, the format required by MS-Cluster and downstream
    purity computation
  repo: http://proteowizard.sourceforge.net/download.html
- name: Python 3.8
  role: Runtime for the Clustering_benchmark_MS_RT.py script and dependency environment
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:82da965c82f2a0ba0f3b48ca54347830381514ab3f3319fde88554395fa32f06
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DB-Search
  - MS-Cluster
  - MS-RT
  - ProteoWizard
  - Python
  - Python 3.8
  - msconvert
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cluster-agreement-statistics-reproduction

## Summary

Reproduces the per-cluster purity agreement between the MS-RT method and a database-search (DB-Search) reference on MS-Cluster output, computing the fraction of clusters within a 10% purity difference and an R2 along the y=x line.

## When to use

Use when you have MS-Cluster clusterinfo.tsv clusters for a ProteomeXchange dataset plus a parallel DB-Search purity estimate, and you need to quantify how well MS-RT purity matches a database-search ground truth at the per-cluster level.

## When NOT to use

- The dataset is metabolomics-only with no parallel database search identifications — in that case MS-RT must be applied directly as the evaluation method rather than validated against DB-Search, since 'in mass spectrometry-based metabolomics, FDR controlled identifications are not currently practical'.
- The clustering tool output lacks the required '#RetTime' column (e.g. Falcon or MaRaCluster output that has not been merged with retention time), so MS-RT purity cannot be computed as-is.

## Inputs

- MS-Cluster clusterinfo.tsv with columns '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx'
- mzML-converted mass spectrometry raw files (e.g. from ProteoWizard msconvert or ThermoRawFileParser)
- Per-cluster purity values from DB-Search (MS-GF+ with FDR-controlled identifications) for the same clusters
- Python 3.8+ virtual environment with dependencies from requirement.txt (Clustering_benchmarking)

## Outputs

- Per-cluster purity comparison table (MS-RT vs DB-Search purity for each of the ~11,152 clusters)
- Agreement statistic: fraction of clusters with purity difference ≤10% (target 85.4%)
- R2 regression statistic along the y=x line (target 0.783)

## How to apply

Run MS-Cluster (e.g. via the GNPS 2.0 Classical Networking Workflow) so clusterinfo.tsv carries '#Filename', '#Scan', '#ParentMass', '#RetTime', and '#ClusterIdx' columns, then evaluate per-cluster purity with MS-RT using src/Clustering_benchmark_MS_RT.py with parameters -c <mscluster_cluster_info.tsv> -t 109333 -methods mscluster -tol 0.1. In parallel, derive per-cluster purity from database search identifications (MS-GF+ with FDR control), the established proteomics evaluation reference that MS-RT is validated against. For each cluster, compare the MS-RT-derived purity to the DB-Search-derived purity: the reproduction target for PXD023047 is ~85.4% of the 11,152 clusters within a 10% purity difference and an R2 of 0.783 along the y=x line (Figure 2.a). Agreement within 10% purity difference is the key decision threshold, indicating MS-RT is a practical substitute for DB-Search where FDR-controlled identifications are not available.

## Related tools

- **MS-Cluster** (Produces the clustered spectra (clusterinfo.tsv) whose per-cluster purity is evaluated by both MS-RT and DB-Search) — https://gnps2.org/workflowinput?workflowname=classical_networking_workflow
- **MS-RT** (Computes per-cluster purity from MS1 retention time consistency within clusters) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **DB-Search** (Provides the reference per-cluster purity from MS-GF+ FDR-controlled identifications for validation)
- **msconvert** (Converts vendor raw files to mzML, the format required by MS-Cluster and downstream purity computation) — http://proteowizard.sourceforge.net/download.html
- **Python 3.8** (Runtime for the Clustering_benchmark_MS_RT.py script and dependency environment) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c data/mscluster/mscluster_cluster_info.tsv -t 109333 -methods mscluster -tol 0.1
```

## Evaluation signals

- The clusterinfo.tsv input contains all required columns: '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx'.
- The per-cluster comparison covers all ~11,152 MS-Cluster clusters of PXD023047 (none dropped silently).
- Approximately 85.4% of clusters (9,524 of 11,152) show MS-RT vs DB-Search purity difference ≤10%.
- R2 of the MS-RT vs DB-Search purity scatter along y=x is ≈0.783, matching Figure 2.a.
- The script was run with the exact parameters: -c <mscluster_cluster_info.tsv> -t 109333 -methods mscluster -tol 0.1.

## Limitations

- The validation target (85.4% agreement, R2 = 0.783) is specific to PXD023047 clustered with MS-Cluster; agreement may differ for other datasets or clustering tools (Falcon, MaRaCluster).
- DB-Search purity requires FDR-controlled identifications, which are not practical in metabolomics — this validation path is proteomics-only.
- The comparison depends on retention time being present in the clusterinfo.tsv; MS/MS spectra lacking retention time cannot contribute to MS-RT purity.

## Evidence

- [results] Agreement target with DB-Search: [evidence span withheld — anchor status: not_found]
- [intro] MS-RT rationale vs DB-Search: "we introduce a retention time-based method, called MS-RT, to assess the correctness of MS/MS clustering in metabolomics."
- [abstract] Validation against DB-Search: "we validate MS-RT by comparing MS-RT to established proteomics clustering evaluation approaches that utilize database search identifications"
- [methods] MS-RT invocation parameters: [evidence span withheld — anchor status: not_found]
- [intro] Metabolomics lacks FDR identifications: "However, in mass spectrometry-based metabolomics, FDR controlled identifications are not currently practical"
- [readme] Required clusterinfo columns: "'#Filename': `#Filename`
- `'scan'`: `#Scan`
- `'mass'`: `#ParentMass`
- `'rt_time'`: `#RetTime`
- `'cluster'`: `#ClusterIdx`"
