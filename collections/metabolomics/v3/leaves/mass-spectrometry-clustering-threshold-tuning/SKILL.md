---
name: mass-spectrometry-clustering-threshold-tuning
description: Use when tuning or comparing spectral clustering quality on MS/MS peak
  lists (MGF), e.g. to verify a tool achieves MaRaCluster-level quality on datasets
  like PXD000561 using clustered spectra ratio, incorrect clustering ratio, and completeness.
when_to_use_negative:
- The goal is peptide identification itself (searching spectra against UniProtKB/Swiss-Prot)
  rather than evaluating or tuning the clustering step
- Input is already a consensus/clustered spectral library rather than raw per-scan
  MGF peak lists
- Only a single clustering run is needed with no quality comparison — a plain HyperSpec
  run suffices without the metric computation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: Primary clustering tool being tuned; sweep cluster_alg (dbscan/hc_single/hc_complete/hc_average),
    eps, and hd_dim on GPU
  repo: https://github.com/wh-xu/Hyper-Spec
- name: MaRaCluster
  role: State-of-the-art benchmark; quality reference point for clustered spectra
    ratio at low incorrect clustering ratios
- name: msCRUSH
  role: Benchmark clustering tool over a threshold sweep on the same dataset
- name: falcon
  role: Benchmark clustering tool (hashing-based) included in the quality comparison
- name: ClusterSheep
  role: GPU-based benchmark clustering tool included in the quality comparison
- name: spectra-cluster
  role: Iterative greedy benchmark clustering tool used to build PRIDE-Cluster libraries
- name: MS-Cluster
  role: Iterative greedy benchmark clustering tool included in the quality comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:2e9538bec8b6552446ac23dc0e3db1989776b1992ce9d600e2d131ac7861a808
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# mass-spectrometry-clustering-threshold-tuning

## Summary

Sweep MS/MS spectral clustering thresholds (DBSCAN eps, hierarchical linkage cutoffs, hypervector dimension D) and evaluate clustering quality via clustered spectra ratio, incorrect clustering ratio, and completeness, benchmarking against tools like MaRaCluster, msCRUSH, spectra-cluster, and MS-Cluster.

## When to use

Use when tuning or comparing spectral clustering quality on MS/MS peak lists (MGF), e.g. to verify a tool achieves MaRaCluster-level quality on datasets like PXD000561 using clustered spectra ratio, incorrect clustering ratio, and completeness.

## When NOT to use

- The goal is peptide identification itself (searching spectra against UniProtKB/Swiss-Prot) rather than evaluating or tuning the clustering step
- Input is already a consensus/clustered spectral library rather than raw per-scan MGF peak lists
- Only a single clustering run is needed with no quality comparison — a plain HyperSpec run suffices without the metric computation

## Inputs

- MGF peak files of MS/MS spectra (e.g. draft human proteome Data set-E, PXD000561)
- Spectrum identifications for completeness computation (e.g. MassIVE reanalysis RMSV000000091.3)
- Clustering outputs from benchmark tools (MaRaCluster, msCRUSH, falcon, ClusterSheep, spectra-cluster, MS-Cluster) over threshold sweeps

## Outputs

- CSV/parquet clustering assignments (bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative)
- Completeness, clustered spectra ratio, and incorrect clustering ratio metrics per tool and threshold
- Comparison table of completeness values across the seven tools

## How to apply

Convert/prepare the MGF peak files and run HyperSpec (wh-xu/Hyper-Spec) with the default configuration, sweeping the clustering algorithm (--cluster_alg dbscan, hc_single, hc_complete, hc_average) and threshold (--eps, e.g. 0.2–0.4), optionally varying the hypervector dimension --hd_dim (default 2048; article tested 128–8192). For each setting, compute clustered spectra ratio, incorrect clustering ratio, and completeness (fragmentation of spectra of the same peptide across clusters), noting that as D increases the incorrect clustering ratio and clustered spectra ratio both decrease while completeness trends differ by algorithm (DBSCAN completeness fell 0.8979→0.8615; hierarchical rose 0.8071→0.8406). Enable post-clustering refinement (--refine) and restrict clustering to charges 2 and 3 (--cluster_charges 2 3) as in the paper's examples. Run the six benchmark tools (MaRaCluster, msCRUSH, falcon, ClusterSheep, spectra-cluster, MS-Cluster) over a threshold sweep on the same dataset, align outputs to the same spectra, and compile a comparison table to verify HyperSpec's quality is comparable at each incorrect-clustering-ratio operating point.

## Related tools

- **HyperSpec** (Primary clustering tool being tuned; sweep cluster_alg (dbscan/hc_single/hc_complete/hc_average), eps, and hd_dim on GPU) — https://github.com/wh-xu/Hyper-Spec
- **MaRaCluster** (State-of-the-art benchmark; quality reference point for clustered spectra ratio at low incorrect clustering ratios)
- **msCRUSH** (Benchmark clustering tool over a threshold sweep on the same dataset)
- **falcon** (Benchmark clustering tool (hashing-based) included in the quality comparison)
- **ClusterSheep** (GPU-based benchmark clustering tool included in the quality comparison)
- **spectra-cluster** (Iterative greedy benchmark clustering tool used to build PRIDE-Cluster libraries)
- **MS-Cluster** (Iterative greedy benchmark clustering tool included in the quality comparison)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- Output parquet contains all required columns (bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative) with one row per spectrum
- Completeness falls in the ranges reported in the paper as D varies: DBSCAN ~0.86–0.90, hierarchical ~0.80–0.84
- HyperSpec clusters more spectra than MaRaCluster at low incorrect clustering ratios, and its completeness is comparable to the six benchmark tools in the compiled table
- Incorrect clustering ratio and clustered spectra ratio both decrease as hypervector dimension D increases for both DBSCAN and hierarchical clustering
- Runtime sanity check: clustering of the ~25 million-spectrum draft human proteome dataset completes in minutes (<15 min) with a GPU rather than hours

## Limitations

- Clustering the full PXD000561 dataset requires a GTX 3090-class GPU with larger memory; smaller GPUs may not fit the dataset
- As HV dimension D increases, correctness improves but fewer spectra are clustered, so there is a quality/coverage trade-off to navigate
- A multi-GPU mode is not yet implemented (proposed as future work), and an optimized distance function for comparing spectrum hypervectors could further improve clustering quality

## Evidence

- [results] as the HV dimension D increases, the incorrect clustering ratio and the clustered spectra ratio decreased for both clustering algorithms: "as the HV dimension D increases, the incorrect clustering ratio and the clustered spectra ratio for both clustering algorithms decreased"
- [results] DBSCAN vs hierarchical completeness across D: "the completeness of DBSCAN decreases from 0.8979 to 0.8615, while hierarchical clustering’s completeness is improved from 0.8071 to 0.8406"
- [readme] eps threshold parameter: "The DBSCAN clustering threshold is `eps=0.2` and post-clustering refinement is `enable`."
- [methods] PXD000561 used for quality evaluation: "The largest data set, PXD000561,1 was used for runtime and clustering quality evaluation."
