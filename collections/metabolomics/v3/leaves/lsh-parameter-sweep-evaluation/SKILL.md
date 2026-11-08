---
name: lsh-parameter-sweep-evaluation
description: Use when tuning (m,n) AND/OR-connective settings of an LSH signal/noise
  classifier on TimsTOF LC-IMS-MS frames, to quantify false-positive/false-negative
  rates or the data-reduction vs. retained-precursor trade-off.
when_to_use_negative:
- The data are chemical noise/contaminants that are self-similar — the LSH approach
  would misclassify them as 'true' signal regardless of (m,n).
- Signals have low relative intensity compared to noise peaks — the similarity measure
  underlying LSH drops and such signals can be lost even at permissive settings.
- A peak-level gold-standard ground truth is required for evaluation — none was available
  for real data; vendor-selected DDA peaks are a biased, high-intensity-only reference.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: pyproteolizard-data
  role: 'Data-access layer for TimsTOF .d files: PyTimsDataHandle exposes frame metadata,
    precursor/fragment frame ids, and frame/spectrum/slice views used for the per-frame
    benchmark'
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Implements the LSH hashing of binned mass-axis windows with (m,n) AND/OR-connective
    amplification and the collision-table lookup
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: TensorFlow 2
  role: GPU acceleration for the LSH implementation and Monte-Carlo simulation of
    the hash family across the (m,n) grid
- name: OpenTIMS
  role: Low-level C++ API for TimsTOF data access underlying pyproteolizard-data
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:b22b2a910debff538356afecac3c11ad309a7e16457c0ebb205ed29b47598e7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lsh-parameter-sweep-evaluation

## Summary

Sweeps the augmented locality-sensitive-hashing parameters (m OR-connectives, n AND-connectives) of the mzBucket/proteolizard signal classifier and quantifies the resulting trade-off between data reduction and retention of true signal in LC-IMS-MS raw data, via Monte-Carlo collision-probability surfaces and real DDA-PASEF frame benchmarks.

## When to use

Use when tuning (m,n) AND/OR-connective settings of an LSH signal/noise classifier on TimsTOF LC-IMS-MS frames, to quantify false-positive/false-negative rates or the data-reduction vs. retained-precursor trade-off.

## When NOT to use

- The data are chemical noise/contaminants that are self-similar — the LSH approach would misclassify them as 'true' signal regardless of (m,n).
- Signals have low relative intensity compared to noise peaks — the similarity measure underlying LSH drops and such signals can be lost even at permissive settings.
- A peak-level gold-standard ground truth is required for evaluation — none was available for real data; vendor-selected DDA peaks are a biased, high-intensity-only reference.

## Inputs

- Unzipped Zenodo record 5036526 TimsTOF PRO DDA-PASEF raw data (.d folder) accessed via PyTimsDataHandle
- Randomly selected subset of precursor-ion frames (e.g. 50 frames with frame number > 1500)
- Vendor-selected DDA-PASEF peptide peak annotations (N=6902 in the paper's subset)
- Candidate (m,n) setting list, e.g. {(32,16), (64,32), (32,32)}

## Outputs

- Collision-probability surface P(m,n) vs. cosine similarity (heatmap/plot, matplotlib)
- Per-(m,n) results table of peak-reduction rate and kept-precursor rate (real_data_classification_rates.csv)
- Per-frame classification of peaks as true signal vs. noise

## How to apply

First characterize a candidate setting by Monte-Carlo simulation: implement random-projection hashes h(x)=sign(<x,r>) over random unit vectors, compose m OR-connectives over blocks of n AND-connectives (e.g. (32,16), (64,32), (32,32)), and numerically compute the collision probability P(m,n) as a function of cosine similarity, rendering it as a heatmap/surface. Then benchmark on real data: load TimsTOF PRO DDA-PASEF frames via pyproteolizard-data, cut each frame's mass axis into 10 Da windows offset by half a window length, bin the axis into equally spaced bins, and compute m hashes per window with pyproteolizard-algorithm. Build the second map structure mapping hash values to occurrence counts and call a window 'signal' if any of its hashes appears in the collision table; repeat for each (m,n) in the sweep. Classify each peak by whether a collided window contains it, and report per-frame peak-reduction rate and the fraction of the 6902 vendor-selected DDA-PASEF peptide peaks retained (excluding noise_2 peaks inside signal windows). The key decision point is that low-threshold settings like (32,16) give very high true- and false-positive rates, while stricter (64,32) or (32,32) settings shrink both; the strong-filter regime can halve data points per frame while retaining almost all vendor-selected peaks.

## Related tools

- **pyproteolizard-data** (Data-access layer for TimsTOF .d files: PyTimsDataHandle exposes frame metadata, precursor/fragment frame ids, and frame/spectrum/slice views used for the per-frame benchmark) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Implements the LSH hashing of binned mass-axis windows with (m,n) AND/OR-connective amplification and the collision-table lookup) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **TensorFlow 2** (GPU acceleration for the LSH implementation and Monte-Carlo simulation of the hash family across the (m,n) grid)
- **OpenTIMS** (Low-level C++ API for TimsTOF data access underlying pyproteolizard-data)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('data/experiment.d'); frame = dh.get_frame(1)
```

## Evaluation signals

- All simulated collision probabilities P(m,n) lie in [0,1] and are monotone in both m and n; surface matches the reference plot in Additional file 1 of DOI 10.1186/s12859-022-04833-5.
- At the strong-filter setting (32,32), data points per frame are reduced by at least half while almost all of the 6902 vendor-selected peaks are retained.
- Reduction rate and kept-precursor rate are inversely related across (m,n), with kept-precursor rates ranging roughly between 70% and almost 100%.
- Low-threshold (32,16) yields very high true-positive and false-positive rates; (64,32) significantly improves discrimination relative to (32,16).
- noise_2 peaks inside signal windows are excluded from classification-rate computation, matching the paper's protocol.

## Limitations

- Chemical noise is self-similar and would be classified as 'true' signal by this approach.
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops.
- Vendor-selected DDA masses correspond only to peptides selected for fragmentation, likely stemming from high-intensity signals — a biased reference; no gold-standard peak-level annotation of real data was available.

## Evidence

- [results] By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates.: "By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates."
- [results] That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks.: "That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks."
- [discussion] A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”.: "A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”."
- [results] To facilitate the lookup of collisions, a second map structure is used that maps hash values to their respective number of occurrences.: "To facilitate the lookup of collisions, a second map structure is used that maps hash values to their respective number of occurrences."
- [readme] The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder"
