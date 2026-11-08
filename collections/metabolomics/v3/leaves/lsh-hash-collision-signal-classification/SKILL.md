---
name: lsh-hash-collision-signal-classification
description: Use when raw TimsTOF LC-IMS-MS frames (e.g., DDA-PASEF precursor frames
  from a .d folder) must be denoised or reduced in size while retaining true peptide
  signal, and intensity thresholding is inadequate.
when_to_use_negative:
- The sample is dominated by self-similar chemical noise (repeated contaminants),
  which the LSH approach would misclassify as 'true' signal
- Signals of interest have very low relative intensity to noise peaks — the similarity
  measure underlying the LSH drops and such signals could be lost
- 'A gold-standard peak-level ground truth is required for evaluation: no such annotation
  of real data was available, and vendor DDA-selected masses are a biased (high-intensity,
  fragmented-only) reference'
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-data
  role: 'Data access to TimsTOF raw data: PyTimsDataHandle exposes frames, spectra
    and slices, precursor/fragment frame ids, and raw index/intensity arrays'
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Implements the locality-sensitive hashing used to compute window hash values
    and the collision-based signal classification
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: OpenTIMS
  role: Low-level C++ API underlying data access to Bruker TDF raw files
- name: pybind11
  role: Interfaces the C++ implementation of proteolizard with Python
  repo: https://github.com/pybind/pybind11
- name: TensorFlow 2
  role: Provides GPU acceleration for the LSH implementation
- name: mzBucket
  role: Original LSH-for-MS repository; implementation since shifted into the proteolizard
    framework
  repo: https://github.com/hildebrandtlab/mzBucket
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:4be61a16e543e2a7bce93b0fd14a1cbd844bd419e1eed1d45722f2a8004fede8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - mzBucket
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lsh-hash-collision-signal-classification

## Summary

Classifies mass spectrometry raw-data peaks as signal or noise using locality-sensitive hashing: windows of a binned mass axis are hashed, and a window is called 'signal' if any of its hash values collides in a shared occurrence map, enabling scalable denoising prefiltering of LC-IMS-MS frames.

## When to use

Use when raw TimsTOF LC-IMS-MS frames (e.g., DDA-PASEF precursor frames from a .d folder) must be denoised or reduced in size while retaining true peptide signal, and intensity thresholding is inadequate.

## When NOT to use

- The sample is dominated by self-similar chemical noise (repeated contaminants), which the LSH approach would misclassify as 'true' signal
- Signals of interest have very low relative intensity to noise peaks — the similarity measure underlying the LSH drops and such signals could be lost
- A gold-standard peak-level ground truth is required for evaluation: no such annotation of real data was available, and vendor DDA-selected masses are a biased (high-intensity, fragmented-only) reference

## Inputs

- Bruker TimsTOF .d raw data folder (e.g., Zenodo record 5036526 TimsTOF PRO DDA-PASEF dataset), accessed via PyTimsDataHandle
- Precursor frames (frame number > 1500 recommended; the paper randomly sampled 50 precursor-ion frames)
- Vendor-selected DDA-PASEF peptide peak list (N=6902 annotated masses) as retention reference
- Augmented-LSH parameters (m OR-connectives, n AND-connectives), e.g., (32,16), (64,32), (32,32)

## Outputs

- Per-window signal/noise classification based on hash-collision table lookups
- Per-peak 'true signal' vs 'noise' labels (peaks inside collided windows = true signal)
- Per-frame peak-reduction rate and fraction of vendor-selected precursor peaks retained
- Averaged per-(m,n) results table, e.g., real_data_classification_rates.csv

## How to apply

Load TimsTOF PRO DDA-PASEF raw data with pyproteolizard-data (PyTimsDataHandle over a Bruker .d folder) and select frames to process. Divide each frame's mass axis into overlapping 10 Da windows offset by half a window length, and bin the mass axis into equally spaced bins, since window hash computation requires the binned form. Compute several hash values per window with pyproteolizard-algorithm under an augmented LSH with n AND-connectives and m OR-connectives — test settings (m,n) in {(32,16), (64,32), (32,32)} to trade false-positive against false-negative rates, where (32,32) acts as a strong filter and (32,16) as a weak one. Build a second map structure mapping hash values to their number of occurrences; call a window 'signal' if at least one of its hash values is found in this collision table, otherwise 'noise'. Classify each peak by whether a collided window contains it, then report per-frame data-point reduction and the fraction of vendor-selected DDA-PASEF peaks retained, excluding noise_2 peaks inside signal windows from the rate computation.

## Related tools

- **pyproteolizard-data** (Data access to TimsTOF raw data: PyTimsDataHandle exposes frames, spectra and slices, precursor/fragment frame ids, and raw index/intensity arrays) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Implements the locality-sensitive hashing used to compute window hash values and the collision-based signal classification) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **OpenTIMS** (Low-level C++ API underlying data access to Bruker TDF raw files)
- **pybind11** (Interfaces the C++ implementation of proteolizard with Python) — https://github.com/pybind/pybind11
- **TensorFlow 2** (Provides GPU acceleration for the LSH implementation)
- **mzBucket** (Original LSH-for-MS repository; implementation since shifted into the proteolizard framework) — https://github.com/hildebrandtlab/mzBucket

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('data/5036526/experiment.d'); frame = dh.get_frame(dh.precursor_frames[-1])
```

## Evaluation signals

- At the strong-filter setting (m,n)=(32,32), the amount of data points per frame is reduced by at least half while almost all of the 6902 vendor-selected DDA-PASEF peaks are retained
- Across (m,n) settings, reduction rate and kept-precursor rate are inversely related, spanning roughly 70% to almost 100% on real data
- (32,16) yields very high true-positive AND very high false-positive rates on synthetic data; (64,32) significantly improves classification; (32,32) loses some windows, lowering both true-positive and false-positive rates
- Noise_2 peaks inside signal windows are explicitly excluded when computing classification rates
- Peak labels are consistent with the rule: a peak is 'true' signal iff it lies within a window that collided with any other window

## Limitations

- Chemical noise is self-similar and would be classified as 'true' signal by this approach
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops
- Synthetic data cannot fully reflect the behavior of real data; no gold-standard peak-level ground-truth annotation of real data was available
- Vendor-selected DDA peaks are a biased reference: they correspond only to peptides selected for fragmentation and likely stem from high-intensity signals
- Noise peaks within signal windows (noise_2) are not removed and require subsequent processing steps
- The reference-based search is only prototypic: monoisotopic m/z can be shifted by multiples of m_n/z and charge state identification sometimes fails

## Evidence

- [results] collision-table signal classification: "A window in question is called “signal” if at least one of its hash values can be found in the collision table, otherwise it is called “noise”."
- [discussion] peak classification rule: "A single peak in the data set is classified by whether there was a collided window that contains the peak."
- [results] augmented LSH tuning: "By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates."
- [results] real-data reduction finding: "That way it is possible to reduce the amount of data points in a frame by at least half, while retaining almost all vendor-selected peaks."
- [readme] data access stack: [evidence span withheld — anchor status: not_found]
- [discussion] windowing and binning: "The calculation of a window’s hash value requires the mass axis in a binned form with equally spaced bins."
- [results] strong filter setting: "this setting may be employed as a strong filter to reduce large data sets."
- [discussion] noise_2 exclusion: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
