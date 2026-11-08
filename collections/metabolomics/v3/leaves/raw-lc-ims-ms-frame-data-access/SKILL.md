---
name: raw-lc-ims-ms-frame-data-access
description: Use when raw Bruker .d TimsTOF PRO data (TDF format) must be read frame-by-frame
  in Python/C++ to obtain recorded mass axes, scans and intensities before any windowing
  or LSH computation.
when_to_use_negative:
- Input is already a processed/centroided feature table or peak list rather than raw
  Bruker TDF data — frame-level access adds nothing
- Data were not acquired on a Bruker timsTOF instrument with TDF-formatted .d folders
  (e.g. mzML from other vendors)
- The task is protein/peptide identification rather than raw-data slicing — use a
  search engine instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-data
  role: 'Primary library: object-oriented C++ classes with pybind11 Python wrappers
    exposing timsTOF frames, spectra and slices from the .d dataset'
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: Low-level C++ API used for data access underneath proteolizard-data
  repo: https://github.com/michalsta/opentims
- name: pybind11
  role: Used to interface the C++ data-access layer with Python
  repo: https://github.com/pybind/pybind11
- name: C++
  role: Implementation language of the low-level OpenTIMS access layer and proteolizard
    C++ classes
- name: Python
  role: Target language in which frames and metadata are exposed for the data-science
    stack (scikit-learn, tensorflow, pytorch)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:9aeb74124a877f631b5e978f7e696e5c7f78b12347f12d060eb35f92927c98e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - C++
  - OpenTIMS
  - Python
  - mzBucket
  - pybind11
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# raw-lc-ims-ms-frame-data-access

## Summary

Programmatic access to raw LC-IMS-MS frames from Bruker timsTOF TDF datasets using pyproteolizard-data (built on the low-level OpenTIMS C++ API), exposing per-frame scans, m/z and intensity arrays to Python for downstream windowing, binning and hashing.

## When to use

Use when raw Bruker .d TimsTOF PRO data (TDF format) must be read frame-by-frame in Python/C++ to obtain recorded mass axes, scans and intensities before any windowing or LSH computation.

## When NOT to use

- Input is already a processed/centroided feature table or peak list rather than raw Bruker TDF data — frame-level access adds nothing
- Data were not acquired on a Bruker timsTOF instrument with TDF-formatted .d folders (e.g. mzML from other vendors)
- The task is protein/peptide identification rather than raw-data slicing — use a search engine instead

## Inputs

- Bruker timsTOF .d folder in TDF format (e.g. the LC-IMS-MS dataset from Zenodo record 5036526, unzipped into the data folder)
- Frame identifier (integer Id) from the acquisition metadata

## Outputs

- PyTimsDataHandle object with exposed per-frame metadata table (Id, Time, Polarity, ScanMode, MsMsType, NumScans, NumPeaks, MzCalibration, T1, T2, TimsCalibration, AccumulationTime, RampTime)
- dh.precursor_frames and dh.fragment_frames numpy arrays of frame ids
- Per-frame pandas table of scan, m/z, intensity values (raw data view as 1D index and intensity arrays)
- Recorded mass axes of TimsTOF PRO frames available for downstream windowing/binning

## How to apply

Instantiate a PyTimsDataHandle with the path to the Bruker .d folder; on construction it reads the acquisition metadata stored by Bruker software in a sqlite database, giving a per-frame table (Id, Time, MsMsType, NumScans, NumPeaks, calibration parameters, etc.) plus numpy arrays dh.precursor_frames and dh.fragment_frames. Retrieve individual frames via data_handle.get_frame(frame_id), which returns scan, m/z and intensity values as pandas tables or 1D index/intensity arrays. Frames are the smallest unit loaded into memory; a frame corresponds to one retention time with a collection of scans (non-normalized ion-mobility values), each scan holding one mass spectrum. For the LSH workflow in the paper, these raw frames provide the recorded mass axes that are subsequently divided into overlapping 10 Da windows and binned at 0.1 m/z spacing before hash computation. If preferred, opentims or alphatims can be swapped in as the data-access backend and used to feed proteolizard classes.

## Related tools

- **pyproteolizard-data** (Primary library: object-oriented C++ classes with pybind11 Python wrappers exposing timsTOF frames, spectra and slices from the .d dataset) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (Low-level C++ API used for data access underneath proteolizard-data) — https://github.com/michalsta/opentims
- **pybind11** (Used to interface the C++ data-access layer with Python) — https://github.com/pybind/pybind11
- **C++** (Implementation language of the low-level OpenTIMS access layer and proteolizard C++ classes)
- **Python** (Target language in which frames and metadata are exposed for the data-science stack (scikit-learn, tensorflow, pytorch))

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; data_handle = PyTimsDataHandle('path/to/experiment.d'); frame = data_handle.get_frame(1); frame.data()
```

## Evaluation signals

- Construction of PyTimsDataHandle succeeds and its metadata table reports expected per-frame fields (e.g. 918 NumScans, T1 ≈ 25.6819, T2 ≈ 25.308 for the reference dataset)
- get_frame(frame_id) returns a table whose rows carry frame, scan, mz and intensity columns with plausible m/z ranges for TimsTOF precursor/fragment frames
- dh.precursor_frames and dh.fragment_frames partition frame ids consistently with the MsMsType column of the metadata table
- The retrieved mass axes, once divided into 10 Da windows binned at 0.1 m/z, cover the full recorded mass range with a 5 Da relative offset between the two window sets
- Alternative backends (opentims or alphatims) feeding proteolizard classes yield consistent frame contents

## Limitations

- Additional ion-mobility recording adds a dimension that puts pressure on processing speed; frame-level loading is designed for exploration and prototyping, not full-dataset batch throughput
- Only Bruker timsTOF TDF data are supported via opentims; other vendors/formats are out of scope
- Data sparsity of LC-IMS-MS raw data makes some traditional LC-MS/MS processing approaches too slow or unsuited

## Evidence

- [readme] PyTimsDataHandle constructor reads sqlite metadata: [evidence span withheld — anchor status: external_doc]
- [readme] Frames defined as smallest unit: [evidence span withheld — anchor status: external_doc]
- [results] Built on OpenTIMS low-level C++ API: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [discussion] Recorded mass axes are the input to windowing: "The set of windows is created by dividing all recorded mass axes into windows."
- [methods] Raw data from Zenodo record 5036526: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder"
- [readme] Alternative data-access backends: [evidence span withheld — anchor status: external_doc]
