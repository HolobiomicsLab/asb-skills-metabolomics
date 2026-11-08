---
name: timstof-raw-frame-data-access
description: Use when raw Bruker timsTOF PRO .d/TDF LC-IMS-MS data must be loaded
  frame-wise in Python (e.g. to select precursor frames by frame id, bin mass axes,
  or feed hashing algorithms) rather than using vendor software.
when_to_use_negative:
- Input is already peak-picked or a feature table rather than raw TimsTOF TDF data
- Data comes from non-Bruker instruments or non-TDF formats (e.g. mzML-only pipelines),
  where OpenTIMS-based access does not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: pyproteolizard-data (proteolizard-data)
  role: Python/C++ library exposing PyTimsDataHandle, TimsFrame, and MzSpectrum classes
    for frame-level timsTOF data access
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: low-level C++ API for Bruker TDF data access on which proteolizard-data is
    built
- name: pybind11
  role: generates the Python bindings that interface the C++ data-access layer with
    Python
- name: Python
  role: target language in which the data handle and frame tables are consumed
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:88e1f9015b7d299775407e69fed7f93d67efe68694fb7c64577e124579d0f38e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - OpenTIMS
  - Python
  - mzBucket
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  - pyproteolizard-data (proteolizard-data)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# timstof-raw-frame-data-access

## Summary

Programmatic access to Bruker timsTOF raw data (.d/TDF format) by wrapping the low-level OpenTIMS C++ API with pybind11 into the pyproteolizard-data Python package, exposing frames, spectra, slices, and metadata tables for downstream LSH or other raw-data processing.

## When to use

Use when raw Bruker timsTOF PRO .d/TDF LC-IMS-MS data must be loaded frame-wise in Python (e.g. to select precursor frames by frame id, bin mass axes, or feed hashing algorithms) rather than using vendor software.

## When NOT to use

- Input is already peak-picked or a feature table rather than raw TimsTOF TDF data
- Data comes from non-Bruker instruments or non-TDF formats (e.g. mzML-only pipelines), where OpenTIMS-based access does not apply

## Inputs

- Bruker timsTOF raw data (.d folder, TDF format) e.g. from Zenodo record 5036526
- Frame-id selection criteria (e.g. precursor frames with frame number > 1500)

## Outputs

- PyTimsDataHandle with per-frame metadata table (pandas DataFrame)
- Per-frame peak tables with frame, scan, mz, intensity columns
- precursor_frames / fragment_frames id arrays for frame selection

## How to apply

Construct a PyTimsDataHandle with the path to the .d folder; it reads the experiment metadata from the acquisition sqlite database and exposes a per-frame overview table (Id, Time, MsMsType, NumScans, NumPeaks, accumulation/ramp time, etc.). Retrieve frames via data_handle.get_frame(id), which returns a pandas table of frame, scan, mz, intensity rows, and use the handle's convenience arrays precursor_frames and fragment_frames to select unfragmented vs. fragmented frames (e.g. precursor-ion frames with frame number > 1500 for DDA-PASEF reduction studies). Under the hood, proteolizard-data builds on OpenTIMS's low-level C++ API and exposes classes through pybind11; if needed, data access can be swapped for timspy or alphatims and fed into proteolizard classes. Frames are treated as sparse 3D tensors over retention time, ion mobility, and m/z and can be split into spectra or slices for binning, vectorization, or point-cloud operations.

## Related tools

- **pyproteolizard-data (proteolizard-data)** (Python/C++ library exposing PyTimsDataHandle, TimsFrame, and MzSpectrum classes for frame-level timsTOF data access) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (low-level C++ API for Bruker TDF data access on which proteolizard-data is built)
- **pybind11** (generates the Python bindings that interface the C++ data-access layer with Python)
- **Python** (target language in which the data handle and frame tables are consumed)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('path/to/experiment.d'); frame = dh.get_frame(1); frame.data()
```

## Evaluation signals

- data_handle.meta_data returns a populated per-frame overview table with columns such as Id, Time, MsMsType, NumScans, NumPeaks, AccumulationTime
- get_frame(id) returns a pandas table with exactly frame, scan, mz, intensity columns and non-zero NumPeaks for the chosen frame
- dh.precursor_frames and dh.fragment_frames partition frame ids consistently with MsMsType (0 for precursor, non-zero for fragmented frames)
- Build succeeds via cmake with CMAKE_BUILD_TYPE=Release against the cloned --recursive proteolizard-data repository
- Peak counts per frame are on the order of hundreds of thousands (e.g. ~260611 peaks in frame 1), consistent with raw TimsTOF frame content

## Limitations

- proteolizard-data focuses on convenient frame/spectrum-level classes rather than full-dataset indexing, unlike timspy or alphatims; large whole-dataset operations may be slow
- Requires the raw .d folder and its sqlite-derived acquisition metadata; cannot operate on preprocessed/converted formats alone
- Loading a whole frame into memory assumes frames fit as sparse 3D tensors; very large datasets push processing toward distributed approaches (e.g. Spark)

## Evidence

- [readme] proteolizard-data builds on top of the low-level C++ API of opentims: "`proteolizard-data` builds on top of the low-level C++ API of `opentims`, adding a thin layer of C++ classes that 
are exposed to python via bindings created with the excellent [`pybind11`]"
- [readme] PyTimsDataHandle reads acquisition metadata from sqlite: "To fetch data from a bruker timsTOF file, use the `PyTimsDataHandle` object. It receives a path to a bruker .d folder 
as it's only construction argument. On construction, it automatically reads"
- [readme] frame convenience arrays for precursor/fragment selection: "a data handle also stores two numpy arrays, `dh.precursor_frames` and `dh.fragmet_frames`, allowing you
to collectively get all frame ids"
- [readme] raw data described as sparse 3D tensors over RT, ion mobility, mz: "Mass spectrometry datasets created using liquid chromatography combined with Ion-mobility result in raw data best 
described as sparse 3D tensors (multidimensional matrices), having a retention-time,"
- [results] article: OpenTIMS low-level C++ API for data access with pybind11 bridging: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [readme] data access backend is interchangeable with timspy or alphatims: [evidence span withheld — anchor status: external_doc]
