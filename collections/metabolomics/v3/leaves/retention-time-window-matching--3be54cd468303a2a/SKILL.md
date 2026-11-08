---
name: retention-time-window-matching
description: Use when evaluating MS/MS spectral clustering (msCluster, Falcon, MaRaCluster)
  on metabolomics or proteomics LC-MS data that lacks annotated peptide/molecule labels,
  by exploiting precursor m/z and RT consistency instead of database identifications.
when_to_use_negative:
- Proteomics data with FDR-controlled database search identifications available —
  the DB-Search evaluation is the established approach and MS-RT is only the label-free
  analogue/validation target
- Cluster info tables missing required columns (filename, scan, mass, rt_time, cluster)
  or lacking retention time, e.g. MaRaCluster output before merging RT from the GNPS
  2.0 PerScanSummarizer results.csv
- Data without LC retention-time information (e.g. direct infusion MS/MS), where mass-to-RT
  windows cannot be built
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-RT
  role: The retention-time-window matching method itself, implemented in src/Clustering_benchmark_MS_RT.py
    to compute purity, N10, and clustered fraction
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: falcon
  role: One of the clustered spectrum inputs whose cluster_info.tsv is scored by MS-RT
    window matching
  repo: https://github.com/bittremieuxlab/falcon
- name: maracluster
  role: Clustering tool whose output is merged with retention time (from GNPS 2.0
    PerScanSummarizer results.csv) before MS-RT scoring
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: msCluster
  role: Clustering tool run via the GNPS 2.0 Classical Networking Workflow; its nf_output/clustering/clusterinfo.tsv
    is an MS-RT input
- name: Python 3.8
  role: Runtime for the Clustering_benchmarking virtual environment in which the MS-RT
    script is executed with pandas
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:0d77994a5db271e25221ef77df95be8627ba6ce996c0f7d708387cfa0b54e532
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
  tools:
  - MS-RT
  - Python
  - Python 3.8
  - falcon
  - maracluster
  - msCluster
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 20
---

# retention-time-window-matching

## Summary

MS-RT pairs MS/MS spectra by MS1 precursor mass and retention-time tolerance windows to define cluster purity without database-search labels, enabling clustering-quality benchmarking for metabolomics where FDR-controlled identifications are unavailable.

## When to use

Use when evaluating MS/MS spectral clustering (msCluster, Falcon, MaRaCluster) on metabolomics or proteomics LC-MS data that lacks annotated peptide/molecule labels, by exploiting precursor m/z and RT consistency instead of database identifications.

## When NOT to use

- Proteomics data with FDR-controlled database search identifications available — the DB-Search evaluation is the established approach and MS-RT is only the label-free analogue/validation target
- Cluster info tables missing required columns (filename, scan, mass, rt_time, cluster) or lacking retention time, e.g. MaRaCluster output before merging RT from the GNPS 2.0 PerScanSummarizer results.csv
- Data without LC retention-time information (e.g. direct infusion MS/MS), where mass-to-RT windows cannot be built

## Inputs

- Post-processed cluster info TSV per tool (mscluster_cluster_info.tsv, falcon_cluster_info.tsv, maracluster_cluster_info.tsv) with columns filename, scan, mass, rt_time, cluster
- Total MS/MS spectra count for normalization (e.g. 109333), passed via -t
- Window tolerance (-tol, default 0.1); RT window 30 s (metabolomics) or 65 s (proteomics); mass window 0.01 m/z

## Outputs

- Per-file MS-RT purity values per clustering tool
- Cluster-average purity metric
- N10 metric and percentage of clustered spectra
- Aggregated ms_rt_benchmark_results.csv comparing msCluster, Falcon, and MaRaCluster

## How to apply

Load the post-processed cluster info table (e.g. falcon_cluster_info.tsv) into pandas and verify it has the columns filename, scan, mass (precursor m/z), rt_time, and cluster. Build MS-RT mass-to-retention-time windows per LC-MS file using a 0.01 m/z window with a 30 s RT window for metabolomics (65 s RT window for proteomics), controlled via the -tol argument (default 0.1). Within each window, group spectra matched by precursor mass and retention time into connected components; per-cluster purity is the fraction of the cluster's spectra belonging to the largest connected component. From these window matches, compute MS-RT purity per LC-MS file, cluster-average purity, the N10 metric, and the percentage of clustered spectra normalized by the total MS/MS count supplied via -t (109333 in the paper's examples). Run src/Clustering_benchmark_MS_RT.py once per tool and aggregate MS-RT purity, cluster-average purity, N10, and clustered fraction into ms_rt_benchmark_results.csv; on proteomics data, validate against the DB-Search analogue.

## Related tools

- **MS-RT** (The retention-time-window matching method itself, implemented in src/Clustering_benchmark_MS_RT.py to compute purity, N10, and clustered fraction) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **falcon** (One of the clustered spectrum inputs whose cluster_info.tsv is scored by MS-RT window matching) — https://github.com/bittremieuxlab/falcon
- **maracluster** (Clustering tool whose output is merged with retention time (from GNPS 2.0 PerScanSummarizer results.csv) before MS-RT scoring) — https://github.com/statisticalbiotechnology/maracluster
- **msCluster** (Clustering tool run via the GNPS 2.0 Classical Networking Workflow; its nf_output/clustering/clusterinfo.tsv is an MS-RT input)
- **Python 3.8** (Runtime for the Clustering_benchmarking virtual environment in which the MS-RT script is executed with pandas)

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/falcon/falcon_cluster_info.tsv -t 109333 -methods falcon -tol 0.1
```

## Evaluation signals

- Each cluster info TSV loads with the exact required columns: filename, scan, mass, rt_time, cluster
- All three tools yield finite MS-RT purity, N10, and clustered-fraction values (no NaN/inf)
- Computed metrics reproduce the relative benchmarking of msCluster, Falcon, and MaRaCluster reported in the paper's MS-RT vs DB-Search comparison
- Percentage clustered values normalize against the total MS/MS count passed via -t (109333 in the paper's examples)
- On proteomics data (e.g. PXD021518), MS-RT results are consistent with the DB-Search benchmark, as shown by the SI consistency heatmap

## Limitations

- In metabolomics, FDR-controlled identifications are not currently practical, so MS-RT purity is a proxy rather than a direct identification-based ground truth
- The RT window differs by domain: 30 s for metabolomics vs 65 s for proteomics; using the wrong window changes purity estimates
- msCluster results depend on the GNPS 2.0 workflow protocol; any processing protocol is allowed as long as output columns are #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx
- Requires Python 3.8 or newer and the Clustering_benchmarking environment with requirements.txt installed

## Evidence

- [methods] Without annotated labels, MS-RT computes purity per LC-MS file as the fraction of a cluster's spectra belonging to the largest connected component of spectra matched by MS1 precursor mass and retention time tolerance.: [evidence span withheld — anchor status: not_found]
- [methods] verifying each has the required columns filename, scan, mass (precursor m/z), rt_time, and cluster: [evidence span withheld — anchor status: not_found]
