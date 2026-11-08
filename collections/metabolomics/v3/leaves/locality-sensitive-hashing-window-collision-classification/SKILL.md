---
name: locality-sensitive-hashing-window-collision-classification
description: Use when raw Bruker TDF/timsTOF frames (e.g. DDA-PASEF) must be strongly
  reduced (≥50% of data points) before downstream processing while preserving vendor-selected
  precursor peaks, with tunable (m,n) hash parameters.
when_to_use_negative:
- The goal is removing chemical noise/contaminants — the article notes repeated chemical
  noise is self-similar and would be classified as 'true' signal by this approach
- Signals of low relative intensity to noise peaks matter — the LSH similarity measure
  drops and such signals can be lost
- A gold-standard peak-level ground truth is required for rigorous evaluation — the
  article reports no such annotation exists for real data, and vendor-selected DDA
  peaks are a biased (high-intensity, fragmented-only) reference
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3473
tools:
- name: mzBucket
  role: LSH pipeline implementation for mass spectrometry data (data and code repository)
  repo: https://github.com/hildebrandtlab/mzBucket
- name: proteolizard-data (pyproteolizard-data)
  role: 'Data access: thin C++ layer on the low-level OpenTIMS API, exposed to Python
    via pybind11; provides PyTimsDataHandle, frames, spectra and binned/vectorized
    mz representation'
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: proteolizard-algorithm (pyproteolizard-algorithm)
  role: Computes the LSH hash values per window used for collision classification
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: Apache Spark
  role: Distributed execution of window hashing and collision classification for large
    data sets, aggregating per-frame reduction rates
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:e39148a3ad145033f84fe1a553511575b5d47e0bfbcbb5d75b20997061946af5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - Apache Spark
  - OpenTIMS
  - Python
  - mzBucket
  - proteolizard-algorithm (pyproteolizard-algorithm)
  - proteolizard-data (pyproteolizard-data)
  - pybind11
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# locality-sensitive-hashing-window-collision-classification

## Summary

Signal/noise classification for TimsTOF LC-IMS-MS/MS raw data: divide each frame's mass axis into overlapping 10 Da windows, compute augmented LSH hash values (m OR, n AND connectives) per window via mzBucket/pyproteolizard, and label windows and peaks as signal only if their hashes collide with other windows.

## When to use

Use when raw Bruker TDF/timsTOF frames (e.g. DDA-PASEF) must be strongly reduced (≥50% of data points) before downstream processing while preserving vendor-selected precursor peaks, with tunable (m,n) hash parameters.

## When NOT to use

- The goal is removing chemical noise/contaminants — the article notes repeated chemical noise is self-similar and would be classified as 'true' signal by this approach
- Signals of low relative intensity to noise peaks matter — the LSH similarity measure drops and such signals can be lost
- A gold-standard peak-level ground truth is required for rigorous evaluation — the article reports no such annotation exists for real data, and vendor-selected DDA peaks are a biased (high-intensity, fragmented-only) reference

## Inputs

- Bruker timsTOF .d folder (TDF format) with LC-IMS-MS/MS frames, e.g. the Zenodo record 5036526 dataset
- Vendor-selected precursor/fragmentation peak annotations (DDA-PASEF subset, N=6902 peptides in the evaluated subset)
- LSH parameter setting (m,n), e.g. (32,32) for strong filtering or (64,32) for stricter discrimination

## Outputs

- Per-window signal/noise labels derived from the hash collision table
- Per-peak classification (true signal vs. noise) based on containment in a collided window
- Per-frame reduction rates (retained vs. discarded data-point counts), e.g. written to spark_reduction_rates.csv for the Spark variant

## How to apply

Load TimsTOF frames through the OpenTIMS low-level C++ API exposed to Python via pybind11 (pyproteolizard-data, PyTimsDataHandle). Cut each recorded mass axis into 10 Da windows offset by half a window length and bin the mass axis into equally spaced bins, since hashing requires binned mz values. Compute several hash values per window with pyproteolizard-algorithm, using a second map structure from hash values to occurrence counts to facilitate collision lookup; tune the false-positive/false-negative balance via n AND and m OR connectives — the strong-filter setting (m,n)=(32,32) can halve data, while (m,n)=(64,32) improves discrimination on synthetic data. Call a window 'signal' if at least one of its hash values appears in the collision table, otherwise 'noise'; classify each individual peak by whether a collided window contains it. For very large runs, execute window hashing and collision classification as Apache Spark jobs and aggregate per-frame retained vs. discarded data-point counts to compute reduction rates.

## Related tools

- **mzBucket** (LSH pipeline implementation for mass spectrometry data (data and code repository)) — https://github.com/hildebrandtlab/mzBucket
- **proteolizard-data (pyproteolizard-data)** (Data access: thin C++ layer on the low-level OpenTIMS API, exposed to Python via pybind11; provides PyTimsDataHandle, frames, spectra and binned/vectorized mz representation) — https://github.com/theGreatHerrLebert/proteolizard-data
- **proteolizard-algorithm (pyproteolizard-algorithm)** (Computes the LSH hash values per window used for collision classification) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **Apache Spark** (Distributed execution of window hashing and collision classification for large data sets, aggregating per-frame reduction rates)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('path/to/experiment.d'); frame = dh.get_frame(1)
```

## Evaluation signals

- Each frame's data points are reduced by at least half while almost all vendor-selected peaks (N=6902 in the reference subset) are retained
- Reduction rate and precursor-keeping rate move inversely when (m,n) hash parameters change
- Every retained window has at least one hash value present in the collision table (hash→occurrence-count map)
- Per-frame reduction rates (e.g. spark_reduction_rates.csv) are complete and consistent with aggregated retained/discarded counts
- On synthetic data, the chosen (m,n) yields ROC behavior at least comparable to (preferably better than) intensity-threshold/SNR filtering

## Limitations

- Chemical noise is self-similar and would be misclassified as 'true' signal by the LSH approach
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops
- Vendor-selected DDA peaks are a biased reference: only fragmented peptides, likely high-intensity signals; no gold-standard peak-level annotation of real data was available
- Peaks labelled 'noise_2' (noise inside signal windows) are not removed and require subsequent processing steps

## Evidence

- [results] high-threshold strong filter retains vendor peaks: "That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks."
- [results] per-peak classification rule: [evidence span withheld — anchor status: not_found]
- [discussion] collision-table window labeling: "A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”."
- [abstract] parameter tuning of false positives/negatives: "Through appropriate choice of algorithm parameters it is possible to balance false-positive and false-negative rates"
- [methods] data-access stack: "pyproteolizard-data (https://github.com/theGreatHerrLebert/proteolizard-data) for data-access"
- [readme] dataset placement for reproducibility: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
