---
name: spectral-clustering-completeness-quantification
description: Use when benchmarking MS/MS clustering tools (msCluster, Falcon, MaRaCluster)
  on mzML/MGF-derived spectra and you need to report how many spectra end up in usable
  clusters (N10) and the overall clustered fraction.
when_to_use_negative:
- You need labeled, FDR-controlled identification-based evaluation — the MS-RT approach
  exists precisely because metabolomics lacks database-search ground truth, so use
  DB-Search validation only on proteomics data
- Input is raw vendor .raw or unclustered spectra — clustering must already have been
  run (via GNPS 2.0 Classical Networking, Falcon, or MaRaCluster) and results post-processed
  into cluster info TSVs first
- Cluster info files lack the required columns (filename, scan, mass, rt_time, cluster)
  — re-run the post-processing scripts (summarize_results, maracluster_processing)
  before computing metrics
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-RT
  role: Mass-to-retention-time windowing method implemented in Clustering_benchmark_MS_RT.py
    that computes purity, N10, and clustered-fraction metrics from cluster info files
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: Falcon
  role: One of the benchmarked clustering tools whose summarized results (falcon_cluster_info.tsv)
    serve as input to the metric computation
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Benchmarked clustering tool whose output is merged with retention time information
    and evaluated for N10 and purity
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: msCluster
  role: 'Benchmarked clustering tool run via the GNPS 2.0 Classical Networking Workflow,
    producing clusterinfo.tsv with columns #Filename, #Scan, #ParentMass, #RetTime,
    #ClusterIdx'
- name: ThermoRawFileParser
  role: Converts Thermo RAW files to mzML or MGF formats upstream of clustering
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:4eb2d4d9e14fe465a5ee43be769ac94d179888098efb54b8623c46d29cd89d86
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Falcon
  - MS-RT
  - MaRaCluster
  - Python
  - ThermoRawFileParser
  - msCluster
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# spectral-clustering-completeness-quantification

## Summary

Quantifies how completely an MS/MS clustering tool groups spectra in a metabolomics or proteomics dataset by computing the N10 metric and the percentage of clustered spectra from per-cluster cluster-info files, alongside MS-RT purity. It captures the completeness half of clustering quality, complementing purity measures.

## When to use

Use when benchmarking MS/MS clustering tools (msCluster, Falcon, MaRaCluster) on mzML/MGF-derived spectra and you need to report how many spectra end up in usable clusters (N10) and the overall clustered fraction.

## When NOT to use

- You need labeled, FDR-controlled identification-based evaluation — the MS-RT approach exists precisely because metabolomics lacks database-search ground truth, so use DB-Search validation only on proteomics data
- Input is raw vendor .raw or unclustered spectra — clustering must already have been run (via GNPS 2.0 Classical Networking, Falcon, or MaRaCluster) and results post-processed into cluster info TSVs first
- Cluster info files lack the required columns (filename, scan, mass, rt_time, cluster) — re-run the post-processing scripts (summarize_results, maracluster_processing) before computing metrics

## Inputs

- Post-processed cluster info TSV per tool (mscluster_cluster_info.tsv, falcon_cluster_info.tsv, maracluster_cluster_info.tsv) with columns filename, scan, mass, rt_time, cluster
- Total number of MS/MS spectra supplied via the -t argument (e.g. 109333)

## Outputs

- Per-method metrics: MS-RT purity, cluster-average purity, N10, and percentage of clustered spectra
- Aggregated benchmark table ms_rt_benchmark_results.csv

## How to apply

Load the post-processed cluster info TSV for each tool (e.g. mscluster_cluster_info.tsv, falcon_cluster_info.tsv, maracluster_cluster_info.tsv) with pandas in the Python 3.8 Clustering_benchmarking environment, verifying the required columns filename, scan, mass, rt_time, and cluster. For each dataset, build the MS-RT mass-to-retention-time windows using 0.01 m/z and 30 s windows (65 s RT window for proteomics), with tolerance controlled by the -tol argument (default 0.1). Compute the MS-RT purity per LC-MS file as the fraction of a cluster's spectra in the largest connected component matched by MS1 precursor mass and RT tolerance, the cluster-average purity, the DB-Search validation analogue, the N10 metric, and the percentage of clustered spectra normalized by the total MS/MS count passed via -t (109333 in the paper's examples). Run src/Clustering_benchmark_MS_RT.py separately per method and aggregate MS-RT purity, cluster-average purity, N10, and percentage clustered into ms_rt_benchmark_results.csv.

## Related tools

- **MS-RT** (Mass-to-retention-time windowing method implemented in Clustering_benchmark_MS_RT.py that computes purity, N10, and clustered-fraction metrics from cluster info files) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **Falcon** (One of the benchmarked clustering tools whose summarized results (falcon_cluster_info.tsv) serve as input to the metric computation) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Benchmarked clustering tool whose output is merged with retention time information and evaluated for N10 and purity) — https://github.com/statisticalbiotechnology/maracluster
- **msCluster** (Benchmarked clustering tool run via the GNPS 2.0 Classical Networking Workflow, producing clusterinfo.tsv with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx)
- **ThermoRawFileParser** (Converts Thermo RAW files to mzML or MGF formats upstream of clustering) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/falcon/falcon_cluster_info.tsv -t 109333 -methods falcon -tol 0.1
```

## Evaluation signals

- Each method yields finite purity, N10, and clustered-fraction values (no NaN/inf) for msCluster, Falcon, and MaRaCluster
- Computed metrics reproduce the relative benchmarking of the three tools reported by the paper's MS-RT vs DB-Search comparison
- Cluster info TSVs contain all required columns: filename, scan, mass (precursor m/z), rt_time, cluster
- Percentage clustered values are normalized by the total MS/MS count passed via -t (109333 in the paper's examples) and fall between 0 and 100
- Aggregated results file ms_rt_benchmark_results.csv contains rows for all three methods with all four metrics

## Limitations

- Window tolerances are dataset-type dependent: 0.01 m/z and 30 s RT for metabolomics, but a 65 s RT window for proteomics; the -tol argument (default 0.1) controls window tolerance
- N10 and clustered-fraction depend on the accuracy of the -t total spectra count; a wrong total skews the normalized percentage
- MS-RT is a surrogate evaluation — it is validated against DB-Search identifications only in proteomics (e.g. PXD021518) where FDR-controlled identifications exist; metabolomics lacks such ground truth

## Evidence

- [other] Without annotated labels, MS-RT computes purity per LC-MS file as the fraction of a cluster's spectra belonging to the largest connected component of spectra matched by MS1 precursor mass and retention time tolerance.: [evidence span withheld — anchor status: not_found]
- [other] Compute the N10 metric and the percentage of clustered spectra, normalizing by the total number of MS/MS spectra supplied via -t (109333 in the paper's examples).: [evidence span withheld — anchor status: not_found]
- [other] For each dataset, build the MS-RT mass-to-retention-time windows and compute the per-file largest-connected-component purity using the 0.01 m/z and 30 s windows (65 s RT window for proteomics): [evidence span withheld — anchor status: not_found]
- [methods] Run src/Clustering_benchmark_MS_RT.py separately for each method and collect the reported metrics.: [evidence span withheld — anchor status: not_found]
