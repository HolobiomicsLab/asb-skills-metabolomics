---
name: mass-axis-window-binning
description: Use when raw timsTOF (bruker TDF/.d) frames must be preprocessed into
  overlapping mass-axis windows and binned form as the input transformation preceding
  LSH hash-value computation in a signal-classification pipeline.
when_to_use_negative:
- Data is not in a timsTOF-compatible format (e.g. plain mzML without ion-mobility
  frames accessible via OpenTIMS/opentims) — the handle expects a bruker .d folder
- Hash values have already been computed and the task is downstream collision-table
  classification rather than window/bin preprocessing
- A per-peak or per-feature classification result is needed directly; this skill only
  produces the windowed/binned intermediate, not signal/noise labels
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3294
tools:
- name: pyproteolizard-data
  role: data-access layer providing PyTimsDataHandle and frame/spectrum objects over
    bruker .d files
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: OpenTIMS
  role: low-level C++ API used for data access to the recorded mass axes of TimsTOF
    frames
  repo: https://github.com/michalsta/opentims
- name: pybind11
  role: creates the Python bindings for the C++ windowing/binning classes
  repo: https://github.com/pybind/pybind11
- name: mzBucket
  role: publication's reference implementation of the LSH pipeline the window/bin
    module feeds into; development shifted to the proteolizard framework
  repo: https://github.com/hildebrandtlab/mzBucket
- name: Python
  role: target language for the standalone windowing/binning module
- name: C++
  role: implementation layer for fast frame-level windowing and binning exposed via
    pybind11
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_008
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:60796e824e7a215e41b1053165a9780e9c8b2ba334fd373110eb3f32d625ae05
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3294
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# mass-axis-window-binning

## Summary

Divides the recorded mass axes of raw LC-IMS-MS frames (e.g. TimsTOF PRO .d data) into 10 Da windows, adds a second window set offset by half a window length (5 Da) so boundary-straddling features remain covered, and bins each window's mass axis into equally spaced 0.1 m/z bins — the binned form required before locality-sensitive hash computation.

## When to use

Use when raw timsTOF (bruker TDF/.d) frames must be preprocessed into overlapping mass-axis windows and binned form as the input transformation preceding LSH hash-value computation in a signal-classification pipeline.

## When NOT to use

- Data is not in a timsTOF-compatible format (e.g. plain mzML without ion-mobility frames accessible via OpenTIMS/opentims) — the handle expects a bruker .d folder
- Hash values have already been computed and the task is downstream collision-table classification rather than window/bin preprocessing
- A per-peak or per-feature classification result is needed directly; this skill only produces the windowed/binned intermediate, not signal/noise labels

## Inputs

- Bruker timsTOF raw data (.d folder, TDF format) e.g. the Zenodo record 5036526 dataset
- Raw frames (frame id) exposed as mass axes via PyTimsDataHandle / OpenTIMS low-level C++ API
- Window length (10 Da) and relative offset (5 Da = half window length)
- Bin width for the mass axis (0.1 m/z)

## Outputs

- First set of 10 Da windows covering each frame's recorded mass axis
- Second set of windows offset by half a window length (5 Da)
- Equally spaced 0.1 m/z binned mass axis per window, ready for hash-value computation

## How to apply

Access raw frames through pyproteolizard-data (a pybind11-wrapped layer over the low-level OpenTIMS C++ API) by constructing a PyTimsDataHandle on the bruker .d folder, then retrieve individual frames whose mass axes are to be transformed. Divide each recorded mass axis into windows of 10 Da length, producing a first set of windows, and create a second set shifted by half a window length (5 Da) so that isotope patterns or features straddling a window boundary remain covered. Bin the mass axis of every window into equally spaced bins of 0.1 m/z width, since calculation of a window's hash value requires this binned representation. Expose the windowing and binning logic as a standalone Python module, with optional C++/pybind11 acceleration, operating on raw frames; the binned windows are then fed to hash computation (e.g. via pyproteolizard-algorithm). Validate by running the module on a frame from the Zenodo record 5036526 dataset and confirming the two window sets cover the full recorded mass range with a 5 Da relative offset and uniform 0.1 m/z bin spacing.

## Related tools

- **pyproteolizard-data** (data-access layer providing PyTimsDataHandle and frame/spectrum objects over bruker .d files) — https://github.com/theGreatHerrLebert/proteolizard-data
- **OpenTIMS** (low-level C++ API used for data access to the recorded mass axes of TimsTOF frames) — https://github.com/michalsta/opentims
- **pybind11** (creates the Python bindings for the C++ windowing/binning classes) — https://github.com/pybind/pybind11
- **mzBucket** (publication's reference implementation of the LSH pipeline the window/bin module feeds into; development shifted to the proteolizard framework) — https://github.com/hildebrandtlab/mzBucket
- **Python** (target language for the standalone windowing/binning module)
- **C++** (implementation layer for fast frame-level windowing and binning exposed via pybind11)

## Examples

```
from proteolizarddata.data import PyTimsDataHandle; dh = PyTimsDataHandle('path/to/experiment.d'); frame = dh.get_frame(1)
```

## Evaluation signals

- The two window sets jointly cover the full recorded mass range of a test frame with a 5 Da relative offset between sets
- Bin spacing within every window is uniform at 0.1 m/z width
- Frames retrieved through PyTimsDataHandle expose expected columns (frame, scan, mz, intensity) and metadata (NumScans, NumPeaks) before windowing
- Number of windows per mass axis equals approximately (mass-axis span)/10 plus the shifted set, consistent with a half-window offset
- Binned windows from a Zenodo record 5036526 frame are acceptable input to pyproteolizard-algorithm hash computation without further transformation

## Limitations

- The windowing/binning step is only the input transformation for LSH; it does not itself classify signal vs. noise
- A single (unshifted) window set alone would miss features straddling window boundaries, hence the half-window (5 Da) shifted second set is required
-  proteolizard-data is built specifically for the bruker timsTOF TDF format; other vendors' raw formats are not covered by this data path

## Evidence

- [discussion] windows covering full mass range: "The set of windows is created by dividing all recorded mass axes into windows."
- [discussion] binned form required: "The calculation of a window’s hash value requires the mass axis in a binned form with equally spaced bins."
- [results] hash values per window: "These mass axes are then cut into small intervals, called windows. For each window several hash values are computed."
- [readme] opentims data access layer: [evidence span withheld — anchor status: external_doc]
- [readme] PyTimsDataHandle usage: [evidence span withheld — anchor status: external_doc]
- [methods] window offset rationale: [evidence span withheld — anchor status: not_found]
