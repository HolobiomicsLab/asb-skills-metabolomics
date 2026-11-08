---
name: density-based-spectrum-clustering
description: Use when millions of MS/MS spectra (mzML/mzXML/MGF peak files) must be
  grouped into pure peptide clusters to reduce data volume, e.g. clustering a PRIDE
  draft human proteome dataset for efficient downstream reprocessing.
when_to_use_negative:
- Clustering metabolomics or top-down data with default bottom-up proteomics preprocessing
  — the README states min_peaks/min_mz_range should be reduced and defaults 'likely
  need to be adjusted accordingly'
- Spectra lacking reliable precursor m/z values — the method partitions vectors per
  precursor charge into 1 m/z precursor-mass buckets and filters neighbors by precursor
  mass tolerance, so it cannot be applied as-is
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: 'End-to-end spectrum clustering tool: feature hashing, nearest neighbor indexing,
    sparse pairwise distance matrix, and DBSCAN clustering'
  repo: https://github.com/bittremieux/falcon
- name: ThermoRawFileParser
  role: Converts Thermo RAW files (e.g. PRIDE PXD000561) to MGF peak files prior to
    clustering
- name: Faiss
  role: Builds the per-bucket Voronoi-diagram inverted index via k-means for fast
    nearest neighbor searching
- name: Scikit-Learn
  role: Provides the DBSCAN algorithm used to cluster the sparse pairwise distance
    matrix
- name: fastcluster
  role: Hierarchical maximum-linkage splitting of clusters so precursor mass differences
    within a cluster stay within tolerance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:34ae1211721605e06776d28056cb5d26073c7a21d8832734a8a16cfbbf36b39a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - Pyteomics
  - Python 3.8
  - Scikit-Learn
  - ThermoRawFileParser
  - falcon
  - fastcluster
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# density-based-spectrum-clustering

## Summary

Cluster large volumes of MS/MS tandem mass spectra using falcon's density-based (DBSCAN) clustering over a sparse cosine-distance nearest-neighbor matrix, enabling data reduction to cluster representatives for downstream proteomics processing.

## When to use

Use when millions of MS/MS spectra (mzML/mzXML/MGF peak files) must be grouped into pure peptide clusters to reduce data volume, e.g. clustering a PRIDE draft human proteome dataset for efficient downstream reprocessing.

## When NOT to use

- Clustering metabolomics or top-down data with default bottom-up proteomics preprocessing — the README states min_peaks/min_mz_range should be reduced and defaults 'likely need to be adjusted accordingly'
- Spectra lacking reliable precursor m/z values — the method partitions vectors per precursor charge into 1 m/z precursor-mass buckets and filters neighbors by precursor mass tolerance, so it cannot be applied as-is

## Inputs

- Peak files in mzML, mzXML, or MGF format (e.g. MGF files converted from Thermo RAW files with ThermoRawFileParser)
- Precursor mass tolerance setting (e.g. 20 ppm) and clustering parameters (eps, min_samples) supplied as command-line arguments or an INI config file

## Outputs

- falcon.csv: comma-separated file with each MS/MS spectrum and its cluster label on a single line
- falcon.mgf: representative spectra exported for each cluster (with --export_representatives)

## How to apply

Preprocess each spectrum by removing precursor-ion peaks and noise peaks below 10% of the base peak intensity, discarding spectra with fewer than 5 peaks or an m/z range below 250 Da, and square-root transforming peak intensities to de-emphasize dominant peaks. Bin spectra at 0.05 m/z and compress the sparse high-dimensional vectors to 800 dimensions via feature hashing with the 32-bit MurmurHash3 algorithm, which conserves cosine similarity. Per precursor charge, partition vectors into 1 m/z precursor-mass buckets, build a Voronoi-diagram inverted index with k-means in Faiss (up to 65536 cells, up to 32 neighboring cells explored per query), and retrieve the 128 nearest neighbors per spectrum. Filter neighbors whose precursor mass differs by more than 20 ppm and store cosine distances to at most 64 nearest neighbors in a sparse pairwise distance matrix, then cluster with DBSCAN (minimum 2 spectra per dense region). Choose eps to balance purity versus completeness: a cosine distance threshold of 0.05 yields approximately 1% incorrectly clustered spectra on the 25-million-spectrum human draft proteome data; README guidance suggests 0.05–0.15 for pure results, up to 0.30 for aggressive clustering. Postprocess clusters with hierarchical maximum-linkage splitting so pairwise precursor mass differences within a cluster do not exceed the 20 ppm tolerance, then write cluster labels to falcon.csv and export representative spectra to falcon.mgf.

## Related tools

- **falcon** (End-to-end spectrum clustering tool: feature hashing, nearest neighbor indexing, sparse pairwise distance matrix, and DBSCAN clustering) — https://github.com/bittremieux/falcon
- **ThermoRawFileParser** (Converts Thermo RAW files (e.g. PRIDE PXD000561) to MGF peak files prior to clustering)
- **Faiss** (Builds the per-bucket Voronoi-diagram inverted index via k-means for fast nearest neighbor searching)
- **Scikit-Learn** (Provides the DBSCAN algorithm used to cluster the sparse pairwise distance matrix)
- **fastcluster** (Hierarchical maximum-linkage splitting of clusters so precursor mass differences within a cluster stay within tolerance)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- falcon.csv contains a cluster label line for every input MS/MS spectrum
- At cosine distance threshold eps = 0.05, the fraction of incorrectly clustered spectra is approximately 1%, as reported for the 25-million-spectrum draft human proteome dataset
- Cluster representatives are exported to falcon.mgf when --export_representatives is set
- After postprocessing, pairwise precursor mass differences within any cluster do not exceed the precursor tolerance (20 ppm)
- Clusters contain spectra corresponding to only a single peptide (purity), which eps crucially governs per the README

## Limitations

- The eps parameter depends on the spectral characteristics of the data and the configured spectrum preprocessing, and the scaling method (none, square root, logarithm, rank) can influence the cosine threshold
- Nearest neighbor indexing settings (n_probe, n_neighbors, n_neighbors_ann, low_dim) trade clustering time against accuracy; querying fewer neighbors or lists gives slightly less accurate clustering
- The full extent of the effect of chimeric spectra on spectrum clustering and identification remains an open research question
- Default preprocessing settings are intended for bottom-up proteomics data; metabolomics/top-down data need adjusted min_peaks/min_mz_range values

## Evidence
