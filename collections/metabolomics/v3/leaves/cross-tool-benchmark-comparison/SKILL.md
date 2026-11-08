---
name: cross-tool-benchmark-comparison
description: Reference for comparing clustering quality of multiple spectral clustering
  tools on the same MS/MS dataset, using the HyperSpec paper's benchmarking metrics
  and workflow.
when_to_use_negative:
- Input is a single-tool runtime/throughput measurement rather than a multi-tool quality
  comparison
- The dataset is too large for the benchmarking workstation — the HyperSpec README
  notes PXD000561 requires a GTX 3090, and smaller datasets need a GTX 1080Ti
- The goal is tuning HyperSpec's own internal parameters (e.g. hd_dim, eps) rather
  than cross-tool quality comparison
edam_operation: http://edamontology.org/operation_3643
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: Tool under test; also the reference for encoding spectra as binary hypervectors
    and clustering with DBSCAN or hierarchical clustering
  repo: https://github.com/wh-xu/Hyper-Spec
- name: MaRaCluster
  role: Benchmark tool using an optimized similarity metric based on the rarity of
    fragment peaks
- name: msCRUSH
  role: Benchmark tool using locality-sensitive hashing (LSH) to achieve fast clustering
    speeds
- name: falcon
  role: Benchmark tool that converts spectra to low-dimensional vectors using a hashing
    strategy
- name: ClusterSheep
  role: Benchmark tool that offloads computations to a GPU
- name: spectra-cluster
  role: Benchmark tool using an iterative greedy approach to merge similar spectra
- name: MS-Cluster
  role: Benchmark tool using an iterative greedy approach to merge similar spectra
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:36e6faee4ca5d0957af12cec9ef3ee2f53e0a5174802c0c9365fbb6a41495b2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - ClusterSheep
  - HyperSpec
  - MS-Cluster
  - MaRaCluster
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3643
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cross-tool-benchmark-comparison

## Summary

A benchmarking skill for comparing spectral clustering quality across multiple MS/MS clustering tools on the same dataset, as demonstrated by HyperSpec benchmarking against MaRaCluster, msCRUSH, falcon, ClusterSheep, spectra-cluster, and MS-Cluster.

## When to use

Reference for comparing clustering quality of multiple spectral clustering tools on the same MS/MS dataset, using the HyperSpec paper's benchmarking metrics and workflow.

## When NOT to use

- Input is a single-tool runtime/throughput measurement rather than a multi-tool quality comparison
- The dataset is too large for the benchmarking workstation — the HyperSpec README notes PXD000561 requires a GTX 3090, and smaller datasets need a GTX 1080Ti
- The goal is tuning HyperSpec's own internal parameters (e.g. hd_dim, eps) rather than cross-tool quality comparison

## Inputs

- MGF peak files for raw spectra data (e.g. the draft human proteome Data set-E, PXD000561, 25 million MS/MS spectra, 131 GB)
- Clustered outputs from each tool: CSV/parquet files with cluster labels per spectrum (HyperSpec exports a parquet with bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative)
- Spectrum identifications used to compute completeness (e.g. MassIVE reanalysis RMSV000000091.3)

## Outputs

- Comparison table of clustered spectra ratio, incorrect clustering ratio, and completeness for all tools
- Per-tool clustering outputs (cluster labels per spectrum)
- Completeness metric per tool

## How to apply

Run each clustering tool on the same dataset (e.g. the draft human proteome Data set-E, PXD000561) with its clustering threshold swept over a range of values. HyperSpec uses the default DBSCAN or hierarchical clustering configuration on the MGF input files, and each competing tool (MaRaCluster, msCRUSH, falcon, ClusterSheep, spectra-cluster, MS-Cluster) is run with its respective clustering threshold configured to sweep a range. Align all outputs to the same set of spectra, then compute the clustered spectra ratio, the incorrect clustering ratio, and the completeness metric (fragmentation of spectra from the same peptide across multiple clusters). Compile these into a single comparison table; the skill is judged successful when the completeness values are directly comparable across all seven tools, e.g. HyperSpec's completeness comparable to the state-of-the-art tools.

## Related tools

- **HyperSpec** (Tool under test; also the reference for encoding spectra as binary hypervectors and clustering with DBSCAN or hierarchical clustering) — https://github.com/wh-xu/Hyper-Spec
- **MaRaCluster** (Benchmark tool using an optimized similarity metric based on the rarity of fragment peaks)
- **msCRUSH** (Benchmark tool using locality-sensitive hashing (LSH) to achieve fast clustering speeds)
- **falcon** (Benchmark tool that converts spectra to low-dimensional vectors using a hashing strategy)
- **ClusterSheep** (Benchmark tool that offloads computations to a GPU)
- **spectra-cluster** (Benchmark tool using an iterative greedy approach to merge similar spectra)
- **MS-Cluster** (Benchmark tool using an iterative greedy approach to merge similar spectra)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- HyperSpec's completeness metric is directly comparable to (within the range of) the six competing tools' completeness values
- The clustered spectra ratio and incorrect clustering ratio are computed on the same aligned set of spectra for all seven tools
- HyperSpec clusters more spectra than MaRaCluster at low incorrect clustering ratios, and maintains high completeness
- The comparison table includes all seven tools (HyperSpec plus the six benchmarks), not just HyperSpec alone

## Limitations

- Completeness is defined by fragmentation of spectra from the same peptide across multiple clusters; it does not directly measure cluster purity
- Fast tools (msCRUSH, falcon, ClusterSheep) achieve short runtimes but exhibit reduced clustering quality compared to MaRaCluster and msCRUSH
- HyperSpec requires Python 3.8+ with CUDA; PXD000561 requires a GTX 3090 with larger memory, other datasets need a GTX 1080Ti
- Multi-GPU mode is not yet implemented in HyperSpec

## Evidence

- [results] HyperSpec consistently clusters more spectra than MaRaCluster at low incorrect clustering ratios: "Using the draft human proteome Data set-E, we examined the clustering quality using different combinations"
- [methods] Completeness measures fragmentation of spectra from the same peptide across multiple clusters: "The largest data set, PXD000561,1 was used for runtime and clustering quality evaluation."
- [other] Each competing tool is run with its clustering threshold swept over a range of values: [evidence span withheld — anchor status: not_found]
- [readme] HyperSpec exports clustering results as a parquet file with specific metadata fields: "The exported meta data for clustering results are compressed and stored in `parquet` file, which records `bucket`, `precursor_charge`, `precursor_mz`, `identifier`, `scan`, `retention_time`,"
- [readme] GPU requirements for the benchmark dataset: "Clustering for PXD000561 dataset requires GTX 3090 with larger memory"
