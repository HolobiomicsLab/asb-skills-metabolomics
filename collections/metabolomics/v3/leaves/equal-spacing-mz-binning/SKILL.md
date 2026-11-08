---
name: equal-spacing-mz-binning
description: Use when raw LC-IMS-MS frames (e.g. bruker TDF via OpenTIMS) must be
  converted into fixed-width m/z windows and equally spaced bins before hashing, window
  collision analysis, or LSH-based signal classification.
when_to_use_negative:
- Data is not ion-mobility timsTOF raw data (TDF/.d folders) — proteolizard-data is
  built specifically on opentims for bruker timsTOF files
- Downstream analysis does not require hash computation or collision-based window
  classification, so the fixed 10 Da/0.1 m/z binning grid is unnecessary
- The analysis operates on already-processed peak/feature tables rather than raw frame
  mass axes
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-data
  role: Provides PyTimsDataHandle, TimsFrame and MzSpectrum classes for frame-level
    access to raw timsTOF mass axes that are then windowed and binned
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: Low-level C++ API underlying proteolizard-data for raw TDF data access
- name: pybind11
  role: Used to interface the C++ binning/windowing implementation with Python
- name: mzBucket
  role: Repository implementing locality-sensitive hashing for mass spectrometry data,
    including the windowing/binning preprocessing
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-algorithm
  role: Consumes the binned windows to compute locality-sensitive hash values
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:00fe7d33816b39945e81727f79a1a13a248bcee62d79ed89c67076a47a9968a9
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
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# equal-spacing-mz-binning

## Summary

Transforms raw TimsTOF PRO mass axes into an overlapping windowed, uniformly binned representation (10 Da windows with a 5 Da offset, 0.1 m/z bins) that is the required input form for locality-sensitive hash computation in the proteolizard/mzBucket pipeline.

## When to use

Use when raw LC-IMS-MS frames (e.g. bruker TDF via OpenTIMS) must be converted into fixed-width m/z windows and equally spaced bins before hashing, window collision analysis, or LSH-based signal classification.

## When NOT to use

- Data is not ion-mobility timsTOF raw data (TDF/.d folders) — proteolizard-data is built specifically on opentims for bruker timsTOF files
- Downstream analysis does not require hash computation or collision-based window classification, so the fixed 10 Da/0.1 m/z binning grid is unnecessary
- The analysis operates on already-processed peak/feature tables rather than raw frame mass axes

## Inputs

- Raw LC-IMS-MS dataset from Zenodo record 5036526 (bruker .d TDF folder, TimsTOF PRO)
- Individual frames / mass spectra (mz, intensity arrays) retrieved via PyTimsDataHandle
- Window length (10 Da), half-window offset (5 Da), and bin width (0.1 m/z) parameters

## Outputs

- First set of 10 Da windows covering the recorded mass range
- Second set of windows shifted by 5 Da relative to the first set
- Per-window mass axes binned into equally spaced 0.1 m/z bins, ready for hash-value computation

## How to apply

Access raw frames through pyproteolizard-data (built on OpenTIMS's low-level C++ API) using a PyTimsDataHandle on a bruker .d folder, and retrieve each frame's recorded mass axis. Divide each mass axis into windows of 10 Da length, then generate a second, overlapping set of windows shifted by half a window length (5 Da) so features straddling a window boundary remain covered. Bin the mass axis of every window into equally spaced bins of 0.1 m/z width, since the calculation of a window's hash value requires the mass axis in a binned form with equally spaced bins. Expose this windowing/binning logic as a standalone Python module with optional C++/pybind11 acceleration so it can feed the LSH (proteolizard-algorithm) directly.

## Related tools

- **pyproteolizard-data** (Provides PyTimsDataHandle, TimsFrame and MzSpectrum classes for frame-level access to raw timsTOF mass axes that are then windowed and binned) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (Low-level C++ API underlying proteolizard-data for raw TDF data access)
- **pybind11** (Used to interface the C++ binning/windowing implementation with Python)
- **mzBucket** (Repository implementing locality-sensitive hashing for mass spectrometry data, including the windowing/binning preprocessing) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-algorithm** (Consumes the binned windows to compute locality-sensitive hash values) — https://github.com/theGreatHerrLebert/proteolizard-algorithm

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; frame = PyTimsDataHandle('path/to/experiment.d').get_frame(1)
```

## Evaluation signals

- The two window sets (base and 5 Da-shifted) jointly cover the full recorded mass range of the frame
- The second window set is offset from the first by exactly half a window length (5 Da) so boundary-straddling features remain covered
- Bins within each window are equally spaced with a uniform 0.1 m/z width
- Running the module on a frame from the Zenodo 5036526 dataset reproduces the expected window/bin structure, enabling reproducible downstream hash computation

## Limitations

- The binning grid (10 Da windows, 5 Da offset, 0.1 m/z bins) is fixed for this LSH scheme; other resolutions require re-deriving the parameters
- The windowing/binning transformation only prepares data — subsequent LSH classification has known failure modes such as self-similar chemical noise being misclassified as true signal and low relative-intensity signals being lost
- The implementation has been updated and shifted to the proteolizard framework; the standalone mzBucket build instructions now point to pyproteolizard-data and pyproteolizard-algorithm

## Evidence

- [discussion] Overlapping windows with 5 Da offset: [evidence span withheld — anchor status: not_found]
- [discussion] Binning requirement for hashing: "The calculation of a window’s hash value requires the mass axis in a binned form with equally spaced bins."
- [results] Data access layer: "We built on top of the low-level C++ API of OpenTIMS [29] for data access."
- [readme] Data handle construction: [evidence span withheld — anchor status: external_doc]
- [results] Windowing before hashing: "These mass axes are then cut into small intervals, called windows. For each window several hash values are computed."
- [methods] Data provenance: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder"
