---
name: clustering-benchmark-metric-implementation
description: Use when you have cluster info TSVs (filename, scan, mass, rt_time, cluster)
  from msCluster/Falcon/MaRaCluster and need label-free purity, N10, or percent-clustered
  metrics to compare clustering tools.
when_to_use_negative:
- You already have FDR-controlled database search identifications (proteomics ground
  truth) — the DB-Search analogue in src/clustering_eval_compare_DB is the appropriate
  validator instead of label-free MS-RT.
- Your clustering output lacks the aligned schema (filename, scan, mass, rt_time,
  cluster) — re-run the post-processing scripts (src/utility/summarize_results, src/utility/maracluster_processing)
  first.
- You only have raw vendor files and no clustering results yet — clustering must be
  run first (GNPS 2.0 classical networking for msCluster, Falcon, MaRaCluster) before
  metrics can be computed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-RT
  role: Label-free evaluation method whose purity and N10 metrics this skill reimplements
    from cluster outputs
- name: Python 3.8
  role: Runtime for the Clustering_benchmarking virtual environment and the Clustering_benchmark_MS_RT.py
    script (pandas-based)
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: MS-Cluster
  role: Clustering tool whose clusterinfo.tsv (from GNPS 2.0 classical networking)
    supplies one of the three benchmarked inputs
- name: Falcon
  role: Clustering tool whose summarized results (via src/utility/summarize_results)
    supply a benchmarked input
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool whose results are merged with retention time (GNPS 2.0 PerScanSummarizer)
    to form a benchmarked input
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: DB-Search
  role: Proteomics identification-based evaluation used to validate MS-RT via the
    cluster-average purity analogue
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:73c1dc131bc62b4178c101943ef9f737987a3bf58d9514586bc3e702b66a067c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
  tools:
  - DB-Search
  - Falcon
  - MS-Cluster
  - MS-RT
  - MaRaCluster
  - Python
  - Python 3.8
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# clustering-benchmark-metric-implementation

## Summary

Reconstruct the MS-RT evaluation metrics (per-file largest-connected-component purity, cluster-average purity, N10, and percentage of clustered spectra) from post-processed cluster info files, enabling label-free benchmarking of MS/MS clustering tools (msCluster, Falcon, MaRaCluster) on metabolomics data.

## When to use

Use when you have cluster info TSVs (filename, scan, mass, rt_time, cluster) from msCluster/Falcon/MaRaCluster and need label-free purity, N10, or percent-clustered metrics to compare clustering tools.

## When NOT to use

- You already have FDR-controlled database search identifications (proteomics ground truth) — the DB-Search analogue in src/clustering_eval_compare_DB is the appropriate validator instead of label-free MS-RT.
- Your clustering output lacks the aligned schema (filename, scan, mass, rt_time, cluster) — re-run the post-processing scripts (src/utility/summarize_results, src/utility/maracluster_processing) first.
- You only have raw vendor files and no clustering results yet — clustering must be run first (GNPS 2.0 classical networking for msCluster, Falcon, MaRaCluster) before metrics can be computed.

## Inputs

- Post-processed cluster info TSV per tool (mscluster_cluster_info.tsv, falcon_cluster_info.tsv, maracluster_cluster_info.tsv) with columns filename, scan, mass, rt_time, cluster
- Total MS/MS spectrum count for normalization (e.g. 109333), passed via the -t argument
- Window tolerance value via -tol (default 0.1)

## Outputs

- Per-method MS-RT purity, cluster-average purity, N10, and percentage of clustered spectra
- Aggregated comparison table ms_rt_benchmark_results.csv covering msCluster, Falcon, and MaRaCluster

## How to apply

Load each post-processed cluster info file (e.g. mscluster_cluster_info.tsv, falcon_cluster_info.tsv, maracluster_cluster_info.tsv) with pandas in the Python 3.8 'Clustering_benchmarking' environment, verifying the required columns filename, scan, mass (precursor m/z), rt_time, and cluster. For each dataset, build MS-RT mass-to-retention-time windows and compute the per-file purity as the fraction of a cluster's spectra belonging to the largest connected component matched by MS1 precursor mass and retention time tolerance, using 0.01 m/z and 30 s windows (65 s RT window for proteomics), with the window tolerance scaled by the -tol argument (default 0.1). Next compute the cluster-average purity metric (the DB-Search analogue for validation on proteomics data) and the N10 metric plus percentage of clustered spectra, normalizing by the total MS/MS spectra supplied via -t (109333 in the paper's examples). Run src/Clustering_benchmark_MS_RT.py once per method, then aggregate MS-RT purity, cluster-average purity, N10, and percent clustered for msCluster, Falcon, and MaRaCluster into ms_rt_benchmark_results.csv; metrics should reproduce the paper's relative tool benchmarking with finite values.

## Related tools

- **MS-RT** (Label-free evaluation method whose purity and N10 metrics this skill reimplements from cluster outputs)
- **Python 3.8** (Runtime for the Clustering_benchmarking virtual environment and the Clustering_benchmark_MS_RT.py script (pandas-based)) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **MS-Cluster** (Clustering tool whose clusterinfo.tsv (from GNPS 2.0 classical networking) supplies one of the three benchmarked inputs)
- **Falcon** (Clustering tool whose summarized results (via src/utility/summarize_results) supply a benchmarked input) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool whose results are merged with retention time (GNPS 2.0 PerScanSummarizer) to form a benchmarked input) — https://github.com/statisticalbiotechnology/maracluster
- **DB-Search** (Proteomics identification-based evaluation used to validate MS-RT via the cluster-average purity analogue)

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/falcon/falcon_cluster_info.tsv -t 109333 -methods falcon -tol 0.1
```

## Evaluation signals

- Each input cluster info TSV contains all required columns: filename, scan, mass (precursor m/z), rt_time, and cluster.
- Every method (msCluster, Falcon, MaRaCluster) yields finite purity, N10, and clustered-fraction values — NaN/inf indicates a mis-specified window tolerance or malformed input.
- The relative ordering of the three tools on MS-RT purity/N10 reproduces the paper's MS-RT vs DB-Search benchmarking comparison (validated against PXD021518 proteomics data).
- Percent-clustered and N10 denominators match the -t supplied total MS/MS spectra (109333 in the paper's examples).
- Results aggregate cleanly into ms_rt_benchmark_results.csv with one row per method across the four metrics.

## Limitations

- Metabolomics lacks FDR-controlled identifications, so MS-RT replaces (rather than complements) database-search ground truth; its validity is demonstrated only via consistency with DB-Search on proteomics data (e.g. PXD021518).
- Window tolerances are dataset-type dependent: 0.01 m/z and 30 s RT for metabolomics, but a 65 s RT window for proteomics — using the wrong window skews purity.
- The -tol argument (default 0.1) controls window tolerance and must be set deliberately; N10 normalization requires the correct total spectrum count via -t.
- msCluster results require the GNPS 2.0 classical networking workflow output (nf_output/clustering/clusterinfo.tsv) or any protocol producing the exact aligned column names.

## Evidence

- [methods] Without annotated labels, MS-RT computes purity per LC-MS file as the fraction of a cluster's spectra belonging to the largest connected component of spectra matched by MS1 precursor mass and retention time tolerance.: [evidence span withheld — anchor status: not_found]
- [intro] To overcome this limitation, we introduce a retention time-based method, called MS-RT, to assess the correctness of MS/MS clustering in metabolomics.: "we introduce a retention time-based method, called MS-RT, to assess the correctness of MS/MS clustering in metabolomics"
- [abstract] we validate MS-RT by comparing MS-RT to established proteomics clustering evaluation approaches that utilize database search identifications: "we validate MS-RT by comparing MS-RT to established proteomics clustering evaluation approaches that utilize database search identifications"
- [readme] For msCluster, you can use any processing protocol as long as the output file has the following column names: "For msCluster, you can use any processing protocol as long as the output file has the following column names"
- [readme] This project requires Python 3.8 or newer: "Before running the project, ensure you have Python installed on your system. This project requires Python 3.8 or newer."
- [methods] Script Location: src/Clustering_benchmark_MS_RT.py: [evidence span withheld — anchor status: not_found]
