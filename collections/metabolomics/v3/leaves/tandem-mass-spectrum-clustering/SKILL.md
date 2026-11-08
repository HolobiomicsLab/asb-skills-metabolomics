---
name: tandem-mass-spectrum-clustering
description: Use when raw shotgun proteomics MS/MS data (e.g. PRIDE PXD000561-style
  datasets with millions of spectra in mzML, mzXML, or MGF files) must be reduced
  to cluster representatives for more efficient downstream identification.
when_to_use_negative:
- 'When clustering metabolomics or top-down data with default preprocessing: the defaults
  (min_peaks=5, min_mz_range=250, min_mz=101, max_mz=500) are intended for bottom-up
  proteomics and min_peaks/min_mz_range should be reduced for metabolomics data.'
- 'When exact (non-approximate) pairwise spectrum similarities are required: falcon
  approximates cosine distances via feature hashing and nearest neighbor indexes,
  trading a small accuracy cost for speed.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: falcon
  role: 'Spectrum clustering tool: feature hashing, nearest neighbor indexing, sparse
    pairwise cosine distance matrix, DBSCAN clustering; requires Python 3.8+'
  repo: https://github.com/bittremieux/falcon
- name: ThermoRawFileParser 1.2.3
  role: Converts vendor raw files (e.g. from PRIDE PXD000561) to MGF peak files used
    as falcon input
- name: Python 3.8
  role: Implementation language of falcon; also used for tabulating scaling results
    and plotting runtime versus number of spectra with matplotlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:8a8699bae901b31e2f4c666ebc1468ea9a847d57e0e7732abb8452cb5fd93379
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python 3.8
  - ThermoRawFileParser 1.2.3
  - falcon
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tandem-mass-spectrum-clustering

## Summary

Cluster large collections of tandem mass spectra (millions of MS/MS spectra) into groups of similar spectra using falcon, which combines feature hashing, fast nearest neighbor searching on inverted Voronoi indexes, and DBSCAN density-based clustering to avoid exhaustive pairwise comparisons.

## When to use

Use when raw shotgun proteomics MS/MS data (e.g. PRIDE PXD000561-style datasets with millions of spectra in mzML, mzXML, or MGF files) must be reduced to cluster representatives for more efficient downstream identification.

## When NOT to use

- When clustering metabolomics or top-down data with default preprocessing: the defaults (min_peaks=5, min_mz_range=250, min_mz=101, max_mz=500) are intended for bottom-up proteomics and min_peaks/min_mz_range should be reduced for metabolomics data.
- When exact (non-approximate) pairwise spectrum similarities are required: falcon approximates cosine distances via feature hashing and nearest neighbor indexes, trading a small accuracy cost for speed.

## Inputs

- Peak files in mzML, mzXML, or MGF format (e.g. MGF files converted from raw data with ThermoRawFileParser 1.2.3)
- Command-line arguments or INI config file specifying precursor_tol, fragment_tol, eps, and preprocessing settings

## Outputs

- Comma-separated CSV file (e.g. falcon.csv) with each MS/MS spectrum and its cluster label on a single line
- Optional MGF file (e.g. falcon.mgf) containing representative spectra for each cluster
- Optionally, a runtime/memory benchmark table (e.g. scaling_runtime.csv) and runtime-versus-spectra plot across input subsets

## How to apply

Run falcon on MGF/mzML/mzXML peak files with a precursor mass tolerance (e.g. 20 ppm), a fragment mass tolerance in Dalton (e.g. 0.05 Da), and a cosine distance threshold eps (0.05–0.15 for pure clusters, up to 0.30 for aggressive clustering); the tool bins high-resolution spectra into low-dimensional vectors via feature hashing (MurmurHash3), partitions spectra per precursor charge into 1 m/z buckets, builds Voronoi-diagram inverted indexes via k-means, computes a sparse pairwise cosine-distance matrix by nearest neighbor search, and clusters it with DBSCAN. Tune spectrum preprocessing (min_peaks=5, min_mz_range=250, min_mz=101, max_mz=500, optional square-root intensity scaling) before clustering, noting that scaling influences the appropriate eps. Optionally export representative spectra per cluster to an MGF file for downstream processing. Optionally benchmark runtime and memory across random subsets of peak files (e.g. 10 to 2212 files) to verify the near-linear scaling of falcon despite quadratic pairwise-comparison complexity.

## Related tools

- **falcon** (Spectrum clustering tool: feature hashing, nearest neighbor indexing, sparse pairwise cosine distance matrix, DBSCAN clustering; requires Python 3.8+) — https://github.com/bittremieux/falcon
- **ThermoRawFileParser 1.2.3** (Converts vendor raw files (e.g. from PRIDE PXD000561) to MGF peak files used as falcon input)
- **Python 3.8** (Implementation language of falcon; also used for tabulating scaling results and plotting runtime versus number of spectra with matplotlib)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- The runtime-versus-number-of-spectra plot across subsets (10 to 2212 randomly selected peak files, ~25 million spectra total) shows the near-linear scaling reported in the paper's scaling evaluation, with the full run completing within hours on a multi-core node (e.g. two 14-core Intel E5-2680v4 CPUs, 128GB memory).
- Cluster purity: the clustering result contains approximately 1% incorrectly clustered spectra at a cosine distance threshold of 0.05.
- Output CSV contains one line per MS/MS spectrum with a cluster label; representative spectra export as a parseable MGF file.
- falcon generates larger highly pure clusters than alternative tools (MS-Cluster, spectra-cluster, MaRaCluster, msCRUSH), yielding a larger reduction in data volume without loss of relevant information.

## Limitations

- The cosine distance threshold eps crucially governs cluster purity and its ideal value depends on the data's spectral characteristics and preprocessing; values between 0.05 and 0.15 are typical, up to 0.30 for aggressive clustering, and the intensity scaling method influences eps.
- falcon's near-linear runtime comes from avoiding exhaustive pairwise comparisons; pairwise spectrum comparisons themselves scale quadratically with the number of processed spectra.
- How MS/MS spectrum preprocessing influences various similarity measures was not evaluated, and the effect of chimeric spectra on clustering and identification remains an open research question.

## Evidence
