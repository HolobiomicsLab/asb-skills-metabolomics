---
name: overlapping-window-coverage-design
description: Use when preparing raw timsTOF frames (Bruker TDF data accessed via OpenTIMS/pyproteolizard-data)
  for LSH-based signal classification and you must choose window lengths, inter-set
  offsets, and bin widths before hashing.
when_to_use_negative:
- The input mass axis is already binned or already partitioned into windows (e.g.
  feeding a downstream LSH collision-table step) — re-windowing would duplicate or
  invalidate the partition
- The raw data is not Bruker timsTOF TDF format, since the data-access layer (OpenTIMS/pyproteolizard-data)
  is built specifically for the Bruker TDF format
- The goal is per-peak classification using existing collided windows rather than
  constructing the window/bin grid itself
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzBucket
  role: Reference implementation of the mass-axis windowing and binning logic for
    LSH on mass spectrometry data
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-data
  role: Object-oriented C++/pybind11 library exposing TimsTOF raw frames, spectra,
    and slices from Bruker .d folders as input to the windowing step
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: Low-level C++ API used under pyproteolizard-data for programmatic access to
    the recorded mass axes of TimsTOF PRO frames
  repo: https://github.com/michalsta/opentims
- name: pybind11
  role: Binding layer used to interface the C++ windowing/binning code with Python
    so the logic can be exposed as a standalone Python module
  repo: https://github.com/pybind/pybind11
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:3ee4e888c0a72178e67c5f4643ec50e2a9021d0f83e82c950187f888be868014
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - C++
  - OpenTIMS
  - Python
  - mzBucket
  - pybind11
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# overlapping-window-coverage-design

## Summary

Design pattern for partitioning the recorded mass axes of TimsTOF PRO LC-IMS-MS frames into two interleaved sets of 10 Da windows — the second shifted by half a window length (5 Da) — and binning each window's mass axis into equally spaced 0.1 m/z bins, so that features straddling a window boundary remain covered before locality-sensitive hash computation.

## When to use

Use when preparing raw timsTOF frames (Bruker TDF data accessed via OpenTIMS/pyproteolizard-data) for LSH-based signal classification and you must choose window lengths, inter-set offsets, and bin widths before hashing.

## When NOT to use

- The input mass axis is already binned or already partitioned into windows (e.g. feeding a downstream LSH collision-table step) — re-windowing would duplicate or invalidate the partition
- The raw data is not Bruker timsTOF TDF format, since the data-access layer (OpenTIMS/pyproteolizard-data) is built specifically for the Bruker TDF format
- The goal is per-peak classification using existing collided windows rather than constructing the window/bin grid itself

## Inputs

- Raw LC-IMS-MS dataset from Bruker timsTOF (TDF .d folder), e.g. the Zenodo record 5036526 dataset unzipped into the data folder
- Raw frames (frame id, scan, mz, intensity records) retrieved via PyTimsDataHandle from pyproteolizard-data or the low-level OpenTIMS C++ API
- Windowing parameters: window length (10 Da), half-window offset (5 Da), bin width (0.1 m/z)

## Outputs

- First set of 10 Da windows covering the recorded mass axis
- Second set of 10 Da windows offset by 5 Da relative to the first set
- Per-window binned mass axis representations with equally spaced 0.1 m/z bins, ready for hash-value computation

## How to apply

Access raw frames through pyproteolizard-data (built on the low-level OpenTIMS C++ API), which exposes the recorded mass axes of each TimsTOF PRO frame. Divide the mass axis into windows of 10 Da length to produce a first window set. Create a second window set shifted by half a window length (5 Da), so that features straddling a boundary in the first set fall wholly inside a window of the second set — the rationale is that window hashing only sees mass axis content inside each window, so a single non-overlapping partition can lose straddling features. Then bin the mass axis of every window into equally spaced bins of 0.1 m/z width, since the calculation of a window's hash value requires the mass axis in binned form. Expose this windowing and binning logic as a standalone Python module (optionally accelerated via C++/pybind11) that operates on raw frames and feeds the per-window hash computation upstream of the LSH collision table.

## Related tools

- **mzBucket** (Reference implementation of the mass-axis windowing and binning logic for LSH on mass spectrometry data) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-data** (Object-oriented C++/pybind11 library exposing TimsTOF raw frames, spectra, and slices from Bruker .d folders as input to the windowing step) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (Low-level C++ API used under pyproteolizard-data for programmatic access to the recorded mass axes of TimsTOF PRO frames) — https://github.com/michalsta/opentims
- **pybind11** (Binding layer used to interface the C++ windowing/binning code with Python so the logic can be exposed as a standalone Python module) — https://github.com/pybind/pybind11

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; frame = PyTimsDataHandle('path/to/experiment.d').get_frame(1)
```

## Evaluation signals

- Run the module on a frame from the Zenodo 5036526 dataset and confirm the two window sets jointly cover the full recorded mass range
- Verify the second window set is shifted by exactly half a window length (5 Da) relative to the first
- Verify each window's mass axis is binned into equally spaced bins of uniform 0.1 m/z width
- Check that features straddling a boundary in the first window set fall fully inside a window of the shifted set
- Confirm each window yields a binned representation suitable as input to per-window hash-value computation

## Limitations

- Chemical noise is self-similar and would be classified as 'true' signal by the downstream LSH approach, so window/bin coverage alone does not distinguish repeated contaminants from peptide signal
- Signals with low relative intensity to noise peaks can be lost because the similarity measure the LSH is based on drops
- Peaks labelled 'noise_2' (noise peaks inside signal windows) are not removed by the approach and require subsequent processing steps

## Evidence

- [discussion] The set of windows is created by dividing all recorded mass axes into windows.: "The set of windows is created by dividing all recorded mass axes into windows."
- [discussion] Binning is required before hashing each window.: "The calculation of a window’s hash value requires the mass axis in a binned form with equally spaced bins."
- [results] Mass axes are cut into windows and hashed per window.: "These mass axes are then cut into small intervals, called windows. For each window several hash values are computed."
- [readme] Zenodo dataset must be unzipped into the data folder for reproducible runs.: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder"
- [readme] proteolizard-data builds on the low-level OpenTIMS C++ API for timsTOF data access.: [evidence span withheld — anchor status: external_doc]
