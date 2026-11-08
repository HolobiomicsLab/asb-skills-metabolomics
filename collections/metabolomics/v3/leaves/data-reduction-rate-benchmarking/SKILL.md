---
name: data-reduction-rate-benchmarking
description: Use when you need to quantify how much raw mass spectrometry data a filtering/denoising
  step removes, and whether retention of relevant signals (e.g. vendor-selected DDA-PASEF
  precursors) is preserved, as a function of tunable hash parameters.
when_to_use_negative:
- Input is already a reduced/peak-picked feature table rather than raw frame-level
  TimsTOF data — there are no per-frame data points left to reduce or count.
- 'You need peak-level ground-truth classification accuracy: no gold-standard peak-level
  annotation of real data existed, so reduction-rate benchmarking cannot substitute
  for ROC evaluation on synthetic data.'
- 'The goal is to remove self-similar chemical noise: the article notes self-similar
  chemical noise would be classified as ''true'' signal by the LSH approach, so reduction
  rates would not reflect removal of such contaminants.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzBucket
  role: LSH signal-classification pipeline whose per-frame data reduction rate is
    benchmarked
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-data
  role: OpenTIMS-based C++/pybind11 data-access layer for loading TimsTOF frames and
    binning the mass axis
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Computes the LSH hash values per window used for signal/noise classification
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: Apache Spark
  role: Executes window hashing and collision classification as distributed jobs and
    aggregates per-frame reduction counts
- name: OpenTIMS
  role: Low-level C++ API underlying data access to Bruker TDF files
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:13d58aded2f872000e829087c88bdb2121659922ee7b1cbd62216fa85a54d58a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - Apache Spark
  - OpenTIMS
  - Python
  - mzBucket
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# data-reduction-rate-benchmarking

## Summary

Benchmark how strongly an LSH-based signal-classification pipeline reduces TimsTOF raw data by aggregating per-frame retained vs. discarded data-point counts and checking that each frame is reduced by at least half while vendor-selected precursor peaks are retained.

## When to use

Use when you need to quantify how much raw mass spectrometry data a filtering/denoising step removes, and whether retention of relevant signals (e.g. vendor-selected DDA-PASEF precursors) is preserved, as a function of tunable hash parameters.

## When NOT to use

- Input is already a reduced/peak-picked feature table rather than raw frame-level TimsTOF data — there are no per-frame data points left to reduce or count.
- You need peak-level ground-truth classification accuracy: no gold-standard peak-level annotation of real data existed, so reduction-rate benchmarking cannot substitute for ROC evaluation on synthetic data.
- The goal is to remove self-similar chemical noise: the article notes self-similar chemical noise would be classified as 'true' signal by the LSH approach, so reduction rates would not reflect removal of such contaminants.

## Inputs

- Raw TimsTOF .d dataset (Bruker TDF format), e.g. Zenodo record 5036526, unzipped into the data folder
- Vendor-selected precursor/fragment annotations (DDA-PASEF subset, N=6902 vendor-selected peptides as reference peaks)
- LSH parameter setting (m,n), e.g. (32,32) for the strong filter

## Outputs

- spark_reduction_rates.csv with per-frame reduction rates
- Per-frame retained vs. discarded data-point counts aggregated across the Spark cluster
- Precursor/peak retention statistics (fraction of vendor-selected peaks kept)

## How to apply

Load TimsTOF frames via the OpenTIMS C++ API exposed to Python with pybind11 (pyproteolizard-data), split each mass axis into 10 Da windows offset by half a window length, and bin the axis into equally spaced bins before hashing. Compute LSH hashes per window with pyproteolizard-algorithm using the strong-filter setting (m,n)=(32,32), maintaining a second map from hash values to occurrence counts for collision lookup; call a window 'signal' if any of its hashes appears in the collision table, otherwise 'noise'. Classify each peak by whether a collided window contains it, then run the window hashing and collision classification as Apache Spark jobs and aggregate per-frame retained vs. discarded data-point counts across the cluster. Compute the reduction rate for every frame, write them to spark_reduction_rates.csv, and verify each frame's reduction is at least 50% with almost all vendor-selected peaks retained; reduction and precursor-keeping rates are inversely related, so (m,n) choices trade these off.

## Related tools

- **mzBucket** (LSH signal-classification pipeline whose per-frame data reduction rate is benchmarked) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-data** (OpenTIMS-based C++/pybind11 data-access layer for loading TimsTOF frames and binning the mass axis) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Computes the LSH hash values per window used for signal/noise classification) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **Apache Spark** (Executes window hashing and collision classification as distributed jobs and aggregates per-frame reduction counts)
- **OpenTIMS** (Low-level C++ API underlying data access to Bruker TDF files)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; data_handle = PyTimsDataHandle('path/to/experiment.d'); frame = data_handle.get_frame(1)
```

## Evaluation signals

- Each frame's data-point reduction rate in spark_reduction_rates.csv is at least 0.5 (≥ half of data points removed).
- Almost all vendor-selected peaks (the DDA-PASEF reference set, N=6902 peptides in the benchmark subset) are retained per frame.
- Reduction and precursor-keeping rates show the expected inverse relationship when (m,n) is varied.
- Per-frame retained vs. discarded counts sum to the original per-frame data-point counts (no peaks lost in aggregation across the Spark cluster).
- Every peak in a frame is classified: each peak is 'signal' if a collided window contains it, otherwise 'noise', with no unassigned windows (window called 'signal' iff at least one hash value is in the collision table).

## Limitations

- Vendor-selected DDA peaks are a biased reference: they correspond only to peptides selected for fragmentation and likely stem from high-intensity signals, so retention estimates overstate performance on weak signals.
- No gold-standard ground-truth peak-level annotation of real data was available for evaluation.
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops.
- Chemical noise is self-similar and would be classified as 'true' signal by this approach, so it is not removed.
- Synthetic data cannot fully reflect the behavior of real data; ROC-based gains may not transfer.

## Evidence

- [results] reduce per-frame data by at least half: "That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks."
- [discussion] collision table signal definition: "A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”."
- [discussion] peak classification rule: "A single peak in the data set is classified by whether there was a collided window that contains the peak."
- [discussion] Spark proposal: "We propose an implementation using a Big Data framework, such as Apache Spark [31], to facilitate testing on many large data sets"
- [readme] data unzip for reproducibility: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
- [abstract] parameter tuning tradeoff: "Through appropriate choice of algorithm parameters it is possible to balance false-positive and false-negative rates"
