---
name: spectral-clustering-quality-evaluation
description: Use when comparing spectral clustering outputs of MS/MS spectra (e.g.,
  MGF-derived clusters with charge 2-3 precursors) against peptide assignments to
  quantify completeness, incorrect clustering ratio, and clustered spectra ratio.
when_to_use_negative:
- Input is a single clustering result with no peptide identifications available —
  entropy-based completeness cannot be computed without peptide labels.
- The goal is runtime/speed benchmarking rather than clustering quality (HyperSpec's
  speedup claims, e.g., 21 million spectra in 24 min vs. 4 h, are measured separately).
- Spectra have not been preprocessed consistently across tools (e.g., differing precursor-peak
  removal or peak-count limits), which invalidates cross-tool quality comparisons.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Generates the clustering outputs (DBSCAN or hierarchical clustering) whose
    quality is evaluated via completeness and incorrect clustering ratio
  repo: https://github.com/wh-xu/Hyper-Spec
- name: MaRaCluster
  role: Benchmark tool for clustering quality comparison against HyperSpec on PXD000561
- name: msCRUSH
  role: Benchmark LSH-based clustering tool included in the six-tool quality comparison
- name: falcon
  role: Benchmark hashing-based clustering tool included in the quality comparison
- name: ClusterSheep
  role: Benchmark GPU-accelerated clustering tool included in the quality comparison
- name: spectra-cluster
  role: Benchmark iterative greedy clustering tool included in the quality comparison
- name: MS-Cluster
  role: Benchmark iterative greedy clustering tool included in the quality comparison
- name: cuML
  role: Provides GPU DBSCAN used to cluster bucket-wise Hamming distance matrices
    during the evaluation sweeps
- name: fastcluster
  role: Provides hierarchical clustering with complete linkage used in the evaluation
    sweeps
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:f4f458f3af8bdd40b7eab7a8fed0cbc1dc7a8b631daf1a0de178c23ea47f8b2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - ClusterSheep
  - CuPy
  - HyperSpec
  - MS-Cluster
  - MaRaCluster
  - Python
  - ThermoRawFileParser
  - cuML
  - falcon
  - fastcluster
  - graphics processing units
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# spectral-clustering-quality-evaluation

## Summary

Evaluates MS/MS spectral clustering quality using clustered spectra ratio, incorrect clustering ratio, and entropy-based completeness against peptide identifications, as done for HyperSpec on the draft human proteome Data set-E (PXD000561).

## When to use

Use when comparing spectral clustering outputs of MS/MS spectra (e.g., MGF-derived clusters with charge 2-3 precursors) against peptide assignments to quantify completeness, incorrect clustering ratio, and clustered spectra ratio.

## When NOT to use

- Input is a single clustering result with no peptide identifications available — entropy-based completeness cannot be computed without peptide labels.
- The goal is runtime/speed benchmarking rather than clustering quality (HyperSpec's speedup claims, e.g., 21 million spectra in 24 min vs. 4 h, are measured separately).
- Spectra have not been preprocessed consistently across tools (e.g., differing precursor-peak removal or peak-count limits), which invalidates cross-tool quality comparisons.

## Inputs

- Clustering output mapping each MS/MS spectrum to a cluster label (HyperSpec exports a CSV/parquet with bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative)
- Peptide identifications for the same spectra (e.g., MS-GF+ PSMs from MassIVE reanalysis RMSV000000091.3 against UniProtKB/Swiss-Prot human reference proteome)
- MGF peak files of the PXD000561 dataset (or equivalent), preprocessed per HyperSpec defaults (charges 2-3, <=50 peaks, L2-normalized intensities)

## Outputs

- Per-configuration quality table (e.g., completeness_vs_hv_dimension.csv) with columns D, clustering algorithm, completeness, incorrect clustering ratio, clustered spectra ratio
- Completeness-vs-distance-threshold trade-off curves per tool/algorithm for benchmark comparison
- Aggregate comparison table of completeness across tools (HyperSpec vs. MaRaCluster, msCRUSH, falcon, ClusterSheep, spectra-cluster, MS-Cluster)

## How to apply

Cluster the preprocessed spectra (charges 2 and 3) per precursor-m/z bucket using either DBSCAN (cuML, eps in the 0.2-0.45 range) or hierarchical clustering with complete linkage (fastcluster) on the bucket-wise normalized Hamming distance matrix of bit-packed binary hypervectors. Compute completeness as one minus the conditional entropy of the cluster distribution given MS-GF+ peptide labels (from MassIVE reanalysis RMSV000000091.3) divided by the maximum entropy reduction the peptide assignments could provide. Sweep the clustering distance threshold to trace the incorrect clustering ratio vs. clustered spectra ratio trade-off, and optionally sweep hyperparameters such as HD dimension D (128-8192) and quantization level Q (4-64 at D=2048). Validate against reported anchors: DBSCAN completeness decreases from 0.8979 (D=128) to 0.8615 (D=8192) while hierarchical clustering completeness improves from 0.8071 to 0.8406; increasing Q from 4 to 64 lowered DBSCAN completeness from 0.8644 to 0.8595 and its incorrect clustering ratio from 1.41% to 1.29%.

## Related tools

- **HyperSpec** (Generates the clustering outputs (DBSCAN or hierarchical clustering) whose quality is evaluated via completeness and incorrect clustering ratio) — https://github.com/wh-xu/Hyper-Spec
- **MaRaCluster** (Benchmark tool for clustering quality comparison against HyperSpec on PXD000561)
- **msCRUSH** (Benchmark LSH-based clustering tool included in the six-tool quality comparison)
- **falcon** (Benchmark hashing-based clustering tool included in the quality comparison)
- **ClusterSheep** (Benchmark GPU-accelerated clustering tool included in the quality comparison)
- **spectra-cluster** (Benchmark iterative greedy clustering tool included in the quality comparison)
- **MS-Cluster** (Benchmark iterative greedy clustering tool included in the quality comparison)
- **cuML** (Provides GPU DBSCAN used to cluster bucket-wise Hamming distance matrices during the evaluation sweeps)
- **fastcluster** (Provides hierarchical clustering with complete linkage used in the evaluation sweeps)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg hc_complete --cluster_charges 2 3 --eps=0.25 --refine
```

## Evaluation signals

- All completeness values lie within [0, 1] and both algorithm traces (DBSCAN and hierarchical) are present for every swept D.
- DBSCAN completeness decreases from ~0.8979 at D=128 toward ~0.8615 at D=8192, while hierarchical clustering completeness increases from ~0.8071 to ~0.8406.
- For the Q sweep at D=2048, DBSCAN incorrect clustering ratio drops from 1.41% (Q=4) to 1.29% (Q=64) and completeness from 0.8644 to 0.8595; Q=16 is the default trade-off.
- Output table contains one row per swept parameter value with all three metrics (clustered spectra ratio, incorrect clustering ratio, completeness) populated.
- Benchmark table shows HyperSpec clustering more spectra than MaRaCluster at low incorrect clustering ratios, confirming comparable completeness to the six state-of-the-art tools.

## Limitations

- Clustering quality is sensitive to HD generation parameters: as D increases, incorrect clustering ratio and clustered spectra ratio decrease for both algorithms, so a single operating point may not generalize.
- Fast tools trade quality for speed: falcon and ClusterSheep exhibit some reduction in clustering quality compared to MaRaCluster and msCRUSH.
- Evaluation depends on reference identifications; the paper used MS-GF+ searches against the 2016/05/23 UniProtKB/Swiss-Prot human reference proteome, so label quality bounds the metric reliability.

## Evidence

- [results] completeness trend: "the completeness of DBSCAN decreases from 0.8979 to 0.8615, while hierarchical clustering’s completeness is improved from 0.8071 to 0.8406"
- [abstract] quality comparison claim: "HyperSpec produces results with comparable clustering quality as state-of-the-art spectral clustering tools while achieving speedups by orders of magnitude"
- [readme] output schema: "records `bucket`, `precursor_charge`, `precursor_mz`, `identifier`, `scan`, `retention_time`, `cluster`, and `is_representative` information"
- [methods] dataset scope: "The largest data set, PXD000561,1 was used for runtime and clustering quality evaluation."
