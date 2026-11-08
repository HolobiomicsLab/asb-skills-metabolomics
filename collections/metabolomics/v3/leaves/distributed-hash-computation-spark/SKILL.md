---
name: distributed-hash-computation-spark
description: Use when TimsTOF frames are too large for single-node LSH hashing and
  you must classify windows as signal/noise via hash collisions across a cluster,
  then quantify per-frame data reduction.
when_to_use_negative:
- The input is a small dataset or a few frames where single-node/GPU-accelerated hashing
  via the TensorFlow 2 path is sufficient — the article proposes Spark specifically
  for testing on many large datasets
- 'The goal is to remove chemical noise or low-relative-intensity signals: self-similar
  chemical noise would be classified as ''true'' signal, and signals with low relative
  intensity to noise peaks could be lost as the similarity measure drops'
- You need a fully validated reference-search/charge-state assignment — the reference
  search is prototypic, monoisotopic m/z can be shifted, and charge-state identification
  sometimes fails
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Apache Spark
  role: Big Data framework in which the window-hashing and collision-classification
    jobs are executed across a cluster
- name: OpenTIMS
  role: Low-level C++ API used for accessing the TimsTOF raw data that the Spark jobs
    hash
- name: pyproteolizard-data
  role: pybind11-exposed Python data-access layer (PyTimsDataHandle, frames, spectra)
    feeding frames into the hashing jobs
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Implements the LSH hashing of binned windows used in the distributed classification
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: mzBucket
  role: Reference implementation and scripts (data, plots, src/python) for the LSH
    signal-classification pipeline the Spark extension reproduces
  repo: https://github.com/hildebrandtlab/mzBucket
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:f3ac2be9feb8721051a9fe7cc405f438253660c0ef5ab56f5c7d842a9495c48a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - Apache Spark
  - OpenTIMS
  - Python
  - mzBucket
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# distributed-hash-computation-spark

## Summary

Runs mzBucket's LSH window-hashing and collision-based signal/noise classification as Apache Spark jobs over TimsTOF raw data, scaling the strong-filter (m,n)=(32,32) setting to large datasets and aggregating per-frame data-point reduction rates.

## When to use

Use when TimsTOF frames are too large for single-node LSH hashing and you must classify windows as signal/noise via hash collisions across a cluster, then quantify per-frame data reduction.

## When NOT to use

- The input is a small dataset or a few frames where single-node/GPU-accelerated hashing via the TensorFlow 2 path is sufficient — the article proposes Spark specifically for testing on many large datasets
- The goal is to remove chemical noise or low-relative-intensity signals: self-similar chemical noise would be classified as 'true' signal, and signals with low relative intensity to noise peaks could be lost as the similarity measure drops
- You need a fully validated reference-search/charge-state assignment — the reference search is prototypic, monoisotopic m/z can be shifted, and charge-state identification sometimes fails

## Inputs

- Bruker TimsTOF raw data (.d folder) from Zenodo record 5036526, unzipped into the data folder
- LSH parameter setting (m,n) — e.g. the strong filter (32,32)
- Window definition: 10 Da mass-axis windows offset by half a window length, with equally spaced mass bins

## Outputs

- Per-window signal/noise labels derived from the hash-collision table
- Per-peak classification (peak belongs to a collided window = signal)
- spark_reduction_rates.csv: per-frame reduction rates over retained vs. discarded data points

## How to apply

Load TimsTOF frames via the low-level OpenTIMS C++ API exposed to Python with pybind11 (pyproteolizard-data), dividing the mass axis into 10 Da windows offset by half a window length and binning it into equally spaced bins as required by hash computation. Compute several LSH hash values per window with pyproteolizard-algorithm under the strong-filter parameter setting (m,n)=(32,32), maintaining a second map structure that maps hash values to their occurrence counts to facilitate collision lookup. Execute the window-hashing and collision-classification stage as Apache Spark jobs, calling a window 'signal' if at least one of its hash values is found in the collision table and otherwise 'noise', and classifying each peak by whether a collided window contains it. Aggregate per-frame retained vs. discarded data-point counts across the Spark cluster and compute the reduction rate for every frame, writing the results to spark_reduction_rates.csv.

## Related tools

- **Apache Spark** (Big Data framework in which the window-hashing and collision-classification jobs are executed across a cluster)
- **OpenTIMS** (Low-level C++ API used for accessing the TimsTOF raw data that the Spark jobs hash)
- **pyproteolizard-data** (pybind11-exposed Python data-access layer (PyTimsDataHandle, frames, spectra) feeding frames into the hashing jobs) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Implements the LSH hashing of binned windows used in the distributed classification) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **mzBucket** (Reference implementation and scripts (data, plots, src/python) for the LSH signal-classification pipeline the Spark extension reproduces) — https://github.com/hildebrandtlab/mzBucket

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; data_handle = PyTimsDataHandle('path/to/experiment.d')
```

## Evaluation signals

- Each frame's data points are reduced by at least half while almost all of the N=6902 vendor-selected peaks in the DDA-PASEF precursor subset are retained
- spark_reduction_rates.csv exists and contains a reduction rate for every processed frame
- A window is labeled 'signal' only when at least one of its hash values appears in the collision table (map of hash values to occurrence counts); single-occurrence hashes yield 'noise'
- Reduction and precursor-keeping rates behave inversely: tightening (m,n) increases reduction but lowers retention of vendor-selected precursors
- Per-frame retained vs. discarded data-point counts aggregated across the Spark cluster sum to the total data points in each frame

## Limitations

- Chemical noise (repeated contaminants) is self-similar and would be misclassified as a 'true' signal by this approach
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops
- Peaks labeled 'noise_2' (noise within signal windows) are not removed and require subsequent processing steps
- No gold-standard ground-truth peak-level annotation of real data was available; vendor-selected DDA peaks are a biased reference (only fragmented, likely high-intensity peptides)

## Evidence

- [discussion] We propose an implementation using a Big Data framework, such as Apache Spark [31], to facilitate testing on many large data sets: "We propose an implementation using a Big Data framework, such as Apache Spark [31], to facilitate testing on many large data sets"
- [discussion] A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”.: "A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”."
- [results] To facilitate the lookup of collisions, a second map structure is used that maps hash values to their respective number of occurrences.: "To facilitate the lookup of collisions, a second map structure is used that maps hash values to their respective number of occurrences."
- [results] That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks.: "That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks."
- [readme] The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder"
