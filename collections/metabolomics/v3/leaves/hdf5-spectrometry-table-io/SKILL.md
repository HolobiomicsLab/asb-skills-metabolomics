---
name: hdf5-spectrometry-table-io
description: Use when an analysis pipeline must pass large high-dimensional MS peak
  or feature tables between DEIMoS stages (thresholding, peak detection, alignment,
  CCS calibration) and persistent HDF5 intermediates/finals are needed.
when_to_use_negative:
- Input is already an in-memory, post-alignment feature table consumed directly by
  a downstream step — adding an HDF5 round-trip is unnecessary.
- Data is raw instrument vendor output (Agilent 6560, Bruker, Waters) not yet converted
  into DEIMoS's N-dimensional peak/feature schema; conversion/extraction must happen
  first.
- A workflow does not need persistent intermediates (e.g. a single in-process DEIMoS
  call on a small tune mix) — direct DataFrame passing is simpler.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Provides deimos.load, deimos.save, and deimos.threshold for reading/writing
    and post-processing high-dimensional MS tables stored as HDF5 files in the workflow.
  repo: github.com/pnnl/deimos
- name: Python
  role: The language/API in which DEIMoS I/O and stage chaining is performed (Python
    API plus command-line tool).
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:201ae580156f73ba340bbd15cf4163cd287b318799e320f9f02078253a60a307
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# hdf5-spectrometry-table-io

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load, intermediate-store, and save peak/feature tables for high-dimensional mass spectrometry data using the DEIMoS HDF5-based I/O API (deimos.load, deimos.save, deimos.partition), keeping each workflow stage's output on disk as a named HDF5 file (e.g. example_data.h5, example_alignment.h5).

## When to use

Use when an analysis pipeline must pass large high-dimensional MS peak or feature tables between DEIMoS stages (thresholding, peak detection, alignment, CCS calibration) and persistent HDF5 intermediates/finals are needed.

## When NOT to use

- Input is already an in-memory, post-alignment feature table consumed directly by a downstream step — adding an HDF5 round-trip is unnecessary.
- Data is raw instrument vendor output (Agilent 6560, Bruker, Waters) not yet converted into DEIMoS's N-dimensional peak/feature schema; conversion/extraction must happen first.
- A workflow does not need persistent intermediates (e.g. a single in-process DEIMoS call on a small tune mix) — direct DataFrame passing is simpler.

## Inputs

- HDF5 (.h5) files of mass spectrometry peak/feature tables produced by upstream DEIMoS stages (e.g. example_data.h5, example_data_peaks.h5, example_tune_pos.h5, example_tune_neg.h5)
- Named keys identifying tables inside the HDF5 store (e.g. "ms1", "A")
- Optional numeric threshold values for immediate post-load signal-span reduction (e.g. 500, 1000, 100)

## Outputs

- Loaded columnar peak/feature frames ready for thresholding, peak detection, partitioning, or alignment
- Saved .h5 intermediates/finals such as example_data.h5, example_data_peaks.h5, example_alignment.h5, aligned feature tables keyed for later retrieval

## How to apply

Write each DEIMoS workflow stage's result to an .h5 file via deimos.save, then re-load it with deimos.load, passing the explicit key for the table you want (e.g. key="ms1" for tune mix data, key="A" for aligned output, or separate keys per data slice). When loading, supply the columns of interest (e.g. ms1 = deimos.load("example_data.h5", key="ms1") for tune mix or deimos.load("example_alignment.h5", key="A") for aligned features) so downstream steps receive a typed, columnar frame rather than raw signal. Apply deimos.threshold immediately after load to reduce signal span (e.g. nominal threshold of 500 on ms1, 1000 on ms1 peaks, or 100 on ms1/ms2), then feed the frame into peak detection, partitioning, alignment, or CCS calibration. Use these HDF5 intermediates as the handoff point between samples so that feature detection, alignment, CCS calibration, and MS2 deconvolution operate on consistent column names and coordinates (m/z, drift time, retention time). Rationale: DEIMoS is 'largely agnostic to acquisition instrumentation' and N-dimensional, so a single HDF5 schema lets Agilent 6560, Bruker, and Waters outputs flow through the same API without format conversion.

## Related tools

- **DEIMoS** (Provides deimos.load, deimos.save, and deimos.threshold for reading/writing and post-processing high-dimensional MS tables stored as HDF5 files in the workflow.) — github.com/pnnl/deimos
- **Python** (The language/API in which DEIMoS I/O and stage chaining is performed (Python API plus command-line tool).)

## Examples

```
ms1 = deimos.load("example_data.h5", key="ms1"); ms1 = deimos.threshold(ms1, threshold=500)
```

## Evaluation signals

- Loaded frame exposes expected columns (m/z, drift time, retention time, intensity) consistent with the DEIMoS N-dimensional schema.
- deimos.load with the intended key (e.g. key="ms1", key="A") returns a non-empty, correctly keyed table rather than the wrong slice.
- Post-load deimos.threshold reduces signal span (e.g. ms1 thresholded at 500, ms1_peaks at 1000, ms1/ms2 at 100) and downstream steps still find features.
- Saved .h5 intermediates round-trip: writing via deimos.save then re-loading yields identical values and column dtypes.
- Files used in the article's example pipeline (example_data.h5, example_data_peaks.h5, example_alignment.h5, example_tune_pos.h5, example_tune_neg.h5) are all readable through the same load/save API without format conversion.

## Limitations

- I/O covers DEIMoS's own HDF5 peak/feature tables; vendor-native formats from Agilent 6560, Bruker, or Waters must be converted first.
- Users must supply the correct key (e.g. "ms1" for tune mix, "A" for alignment) — the API does not auto-discover which table in the file is wanted.
- Large file sizes are expected; the article notes 912 data files cumulatively 1.1 terabytes for the 112 study samples plus 40 QC and 10 blanks, so disk and I/O time are nontrivial.

## Evidence

- [results] Named HDF5 keys per table: "data["A"] = deimos.load("example_alignment.h5", key="A")"
- [results] Load-and-threshold pattern on loaded tables: "ms1 = deimos.threshold(ms1, threshold=500)"
- [results] Threshold applied to peak-detected output file: "ms1_peaks = deimos.threshold(ms1_peaks, threshold=1000)"
- [results] Tune mix loaded with explicit key: "tune_pos = deimos.load("example_tune_pos.h5", key="ms1")"
- [readme] N-dimensional, instrumentation-agnostic data model: "Notably, DEIMoS operates on N-dimensional data, largely agnostic to acquisition instrumentation"
- [readme] Pipeline stage outputs characterized by aligned features across samples: "with the output comprising detected features aligned across study samples and characterized by mass, CCS, tandem mass spectra, and isotopic signature."
