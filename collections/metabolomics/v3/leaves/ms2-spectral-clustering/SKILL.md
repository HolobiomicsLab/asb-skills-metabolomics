---
name: ms2-spectral-clustering
description: Use when tens of millions of MS/MS spectra (tens to hundreds of GB) must
  be merged into consensus spectra before peptide identification and CPU-only clustering
  tools like MS-Cluster, spectra-cluster, or MaRaCluster are too slow.
when_to_use_negative:
- The data set is small enough that CPU-only tools (MaRaCluster, spectra-cluster)
  finish in acceptable time — HyperSpec's GPU requirement (CUDA, GTX 1080Ti/3090)
  is unnecessary overhead.
- No CUDA-capable GPU is available and the full PXD000561-scale clustering is needed,
  since the README states clustering PXD000561 'requires GTX 3090 with larger memory'.
- Precise peptide-level clustering quality is the sole priority and the trade-offs
  of fast tools (e.g., reduced quality seen for falcon and ClusterSheep) are unacceptable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: GPU-accelerated hyperdimensional-clustering library executing the five-stage
    pipeline (preprocessing, bucket division, HD encoding, HD distance computation,
    clustering) over MGF input
  repo: https://github.com/wh-xu/Hyper-Spec
- name: ThermoRawFileParser
  role: Converts PRIDE raw files (e.g., PXD000561) to MGF format, retaining charge
    2 and 3 spectra, before clustering
- name: MaRaCluster
  role: CPU-only state-of-the-art reference tool used as the runtime baseline (>4
    h) that HyperSpec must outperform
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:fbf8601fb6ff2213c0e37186cc5e6a15dac3b415b64d95bbcfa65d23cab9275f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - MaRaCluster
  - Python
  - ThermoRawFileParser
  - graphics processing units
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ms2-spectral-clustering

## Summary

Cluster large-scale MS/MS spectra (e.g., >21M spectra from PXD000561) by encoding them as binary hypervectors via hyperdimensional computing and clustering buckets on GPU with HyperSpec, cutting runtime from hours (~4 h with MaRaCluster) to ~24 min.

## When to use

Use when tens of millions of MS/MS spectra (tens to hundreds of GB) must be merged into consensus spectra before peptide identification and CPU-only clustering tools like MS-Cluster, spectra-cluster, or MaRaCluster are too slow.

## When NOT to use

- The data set is small enough that CPU-only tools (MaRaCluster, spectra-cluster) finish in acceptable time — HyperSpec's GPU requirement (CUDA, GTX 1080Ti/3090) is unnecessary overhead.
- No CUDA-capable GPU is available and the full PXD000561-scale clustering is needed, since the README states clustering PXD000561 'requires GTX 3090 with larger memory'.
- Precise peptide-level clustering quality is the sole priority and the trade-offs of fast tools (e.g., reduced quality seen for falcon and ClusterSheep) are unacceptable.

## Inputs

- MGF peak files generated from PRIDE raw files (e.g., PXD000561, 131 GB draft human proteome MS data)
- CLI options: hd_dim, hd_Q, cluster_charges (2 3), cluster_alg (dbscan, hc_single, hc_complete, hc_average), eps, use_gpu_cluster

## Outputs

- CSV clustering result file mapping each MS/MS spectrum to its cluster label (runtime_report.csv for the timing benchmark)
- Compressed parquet metadata with bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, and is_representative fields
- Optionally exported representative (consensus) MGF spectra via --representative_mgf

## How to apply

Download raw files from PRIDE (e.g., PXD000561) and convert them to MGF with ThermoRawFileParser, retaining precursor charge 2 and 3 spectra. Run the five-stage HyperSpec pipeline (`python src/main.py <mgf_folder> output.csv`) with defaults hd_dim=2048, hd_Q=16, cluster_charges 2 3, cluster_alg hc_complete, eps=0.4, use_gpu_cluster enabled; note the README requires a GTX 3090 for PXD000561-scale data. Preprocessing runs in parallel on k CPU cores and applies fixed filters: remove precursor-related peaks and peaks below 1% of base-peak intensity, drop spectra with <5 peaks or <250 Da mass range, keep at most the 50 highest-intensity peaks, and L2-normalize intensities to [0,1]. Spectra are sorted by ascending precursor m/z into buckets, batch-encoded (batch_size=5000) into binary hypervectors on GPU, and each bucket is clustered by normalized Hamming distance using DBSCAN or hierarchical clustering with post-clustering refinement. Measure wall-clock time from preprocessing start to clustering finish (excluding consensus generation) and compare against the ~24 min reference for >21M spectra versus the >4 h MaRaCluster runtime.

## Related tools

- **HyperSpec** (GPU-accelerated hyperdimensional-clustering library executing the five-stage pipeline (preprocessing, bucket division, HD encoding, HD distance computation, clustering) over MGF input) — https://github.com/wh-xu/Hyper-Spec
- **ThermoRawFileParser** (Converts PRIDE raw files (e.g., PXD000561) to MGF format, retaining charge 2 and 3 spectra, before clustering)
- **MaRaCluster** (CPU-only state-of-the-art reference tool used as the runtime baseline (>4 h) that HyperSpec must outperform)

## Examples

```
python src/main.py ~/dataset/ ./output.csv  --cpu_core_preprocess=4 --cluster_alg hc_complete --cluster_charges 2 3 --eps=0.25 --refine
```

## Evaluation signals

- End-to-end wall-clock runtime (preprocessing start to clustering finish, excluding consensus generation) is on the order of ~24 min for >21M spectra of the 131 GB PXD000561 data set.
- Runtime is far below the >4 h reference reported for MaRaCluster on the same data.
- Output CSV assigns exactly one cluster label per input MS/MS spectrum; parquet metadata contains all required fields (bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative).
- Clustering quality (completeness, incorrect clustering ratio, clustered spectra ratio) is comparable to state-of-the-art tools, and peptide identification on consensus spectra matches state-of-the-art tools.
- Only charge 2 and 3 spectra are clustered, matching the cluster_charges 2 3 setting.

## Limitations

- Clustering the PXD000561 data set requires a GTX 3090 with larger memory; smaller data sets run on GTX 1080Ti, and other NVIDIA GPUs need further testing.
- Spectrum preprocessing contributes 20-90% of overall runtime in state-of-the-art tools and remains a critical bottleneck even with the parallelized preprocessor.
- A multi-GPU mode is not yet implemented; near-linear speedup from multi-GPU integration is proposed as future work.
- No optimized distance function for comparing spectrum hypervectors exists yet; deriving one is proposed to further improve clustering quality.
- Hypervector dimension D trades quality: increasing D from 128 to 8192 decreases both incorrect clustering ratio and clustered spectra ratio, and DBSCAN completeness fell from 0.8979 to 0.8615 as D increased.

## Evidence

- [methods] five-stage pipeline: "The overall data processing flow of HyperSpec consists of five main steps, including spectrum preprocessing, bucket division, hyperdimensional (HD) encoding, HD distance computation, and clustering."
- [methods] preprocessing filters: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [abstract] runtime claim: "HyperSpec produces results with comparable clustering quality as state-of-the-art spectral clustering tools while achieving speedups by orders of magnitude"
- [readme] GPU requirement for PXD000561: "Clustering for PXD000561 dataset requires GTX 3090 with larger memory"
- [readme] environment requirements: "HyperSpec requires `Python 3.8+` with `CUDA` environment. A GPU should be installed properly."
- [readme] runtime benchmark target: "HyperSpec shortens the runtime on the draft human proteome dataset with 25 million spectra from a few hours to <15 minutes."
