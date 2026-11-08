---
name: tims-raw-data-access
description: Use when raw data are Bruker timsTOF .d folders (TDF format) with DDA-PASEF
  acquisitions and you need frame-, scan-, or spectrum-level access to m/z, intensity,
  and ion-mobility values in a Python pipeline.
when_to_use_negative:
- Input is already processed, e.g. a peak/feature table or mzML converted output,
  rather than a Bruker .d TDF folder
- Data come from non-timsTOF instruments without ion mobility, where OpenTIMS/proteolizard-data
  cannot read the vendor format
- Only lightweight high-level access is needed and timspy or alphatims already suffice;
  proteolizard-data explicitly positions itself as an alternative to those backends
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-data
  role: 'Primary data-access library: PyTimsDataHandle, TimsFrame and MzSpectrum classes
    wrapping timsTOF raw data for Python'
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: Low-level C++ API on which pyproteolizard-data is built for TDF data access
  repo: https://github.com/michalsta/opentims
- name: pybind11
  role: Binding layer used to expose the C++ data-access classes to Python
  repo: https://github.com/pybind/pybind11
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:dcf3c9598f4208c3d01dea4b70d8d1aa2114fb210b2cdce5d6fa35a22e46be07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-data
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - pybind11
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# tims-raw-data-access

## Summary

Programmatic access to Bruker timsTOF ion-mobility raw data (TDF format) via pyproteolizard-data, which wraps the low-level OpenTIMS C++ API with pybind11 and exposes frames, scans, and spectra as Python objects. It is the data-loading foundation for downstream LSH signal classification and isotope pattern matching workflows.

## When to use

Use when raw data are Bruker timsTOF .d folders (TDF format) with DDA-PASEF acquisitions and you need frame-, scan-, or spectrum-level access to m/z, intensity, and ion-mobility values in a Python pipeline.

## When NOT to use

- Input is already processed, e.g. a peak/feature table or mzML converted output, rather than a Bruker .d TDF folder
- Data come from non-timsTOF instruments without ion mobility, where OpenTIMS/proteolizard-data cannot read the vendor format
- Only lightweight high-level access is needed and timspy or alphatims already suffice; proteolizard-data explicitly positions itself as an alternative to those backends

## Inputs

- Bruker timsTOF .d folder containing TDF raw data and acquisition metadata SQLite database
- Zenodo record 5036526 dataset (unzipped into the data folder)
- Frame id or precursor/fragment frame id lists

## Outputs

- PyTimsDataHandle object with meta_data overview table per frame (Id, Time, Polarity, ScanMode, MsMsType, NumScans, NumPeaks, accumulation/ramp times)
- pandas tables of frame raw data with columns: frame, scan, mz, intensity
- MzSpectrum / TimsFrame objects and 1D index and intensity arrays per frame or spectrum slice

## How to apply

Construct a PyTimsDataHandle from the path to the .d folder; it reads acquisition metadata from the SQLite database Bruker stores alongside the TDF raw data, exposing frame-level properties (frame id, retention time, polarity, scan mode, MS/MS type, accumulation/ramp time) plus dh.precursor_frames and dh.fragmet_frames arrays. Retrieve individual frames with data_handle.get_frame(id), which returns a pandas table with frame, scan, mz, and intensity columns, and split frames into spectra or raw 1D index/intensity arrays as needed. Frames are the smallest unit loaded into memory and correspond to a single retention time with a collection of scans (non-normalized ion-mobility values), each scan holding one mass spectrum. For the paper's evaluation, use this handle to load the TDF/TIMS subset from Zenodo record 5036526 so unidentified signal windows can be sliced and hashed with pyproteolizard-algorithm's LSH. pyproteolizard-data builds on the low-level C++ OpenTIMS API, exposed to Python via pybind11, and integrates with the Python data science stack (scikit-learn, TensorFlow, pytorch); the C++ layer must be built with CMake in Release mode before use.

## Related tools

- **pyproteolizard-data** (Primary data-access library: PyTimsDataHandle, TimsFrame and MzSpectrum classes wrapping timsTOF raw data for Python) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (Low-level C++ API on which pyproteolizard-data is built for TDF data access) — https://github.com/michalsta/opentims
- **pybind11** (Binding layer used to expose the C++ data-access classes to Python) — https://github.com/pybind/pybind11

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; data_handle = PyTimsDataHandle('path/to/experiment.d'); frame = data_handle.get_frame(1)
```

## Evaluation signals

- data_handle.meta_data returns an overview table with one row per acquired frame, including plausible values for Id, Time, NumScans (e.g. 918) and NumPeaks
- dh.precursor_frames and dh.fragmet_frames partition frame ids consistently with MsMsType (fragment frames show MsMsType 8, the first frame 0)
- get_frame(id).data() yields a pandas table with exactly the columns frame, scan, mz, intensity
- Summed raw-data view arrays (indices/intensities) reconstruct the same peaks as the binned spectrum view for a given frame
- For the paper's evaluation, loading must reproduce the Zenodo 5036526 subset in which the vendor software selected N=6902 peptides for fragmentation

## Limitations

- Requires building the C++ layer with CMake before use; pyproteolizard-data is a thin convenience layer and users may need opentims or alphatims directly for other backends
- Data are sparse 3D tensors (retention time, ion mobility, m/z) and frames are normalized differently per scan type, so downstream algorithms must handle the extra ion-mobility dimension
- proteolizard-data is built to quickly explore timsTOF datasets and prototype ideas, not as a fully hardened production pipeline

## Evidence

- [readme] PyTimsDataHandle object. It receives a path to a bruker .d folder as it's only construction argument. On construction, it automatically reads experiment metadata: [evidence span withheld — anchor status: external_doc]
- [readme] proteolizard-data builds on OpenTIMS C++ API exposed via pybind11: [evidence span withheld — anchor status: not_found]
- [results] OpenTIMS low-level C++ API used for data access in the paper: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [readme] Frames are the smallest loaded unit; a frame is one retention time with scans of ion mobility each holding a mass spectrum: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset must be unzipped into the data folder for reproducible runs: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder to ensure reproducible runs of the scripts."
