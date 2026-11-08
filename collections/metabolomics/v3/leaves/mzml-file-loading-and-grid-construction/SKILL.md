---
name: mzml-file-loading-and-grid-construction
description: Use when raw LC-IMS-MS/MS acquisitions in mzML(.gz) format (e.g. Agilent
  6560 Ion Mobility LC/Q-TOF deposits such as MSV000091746) must be converted into
  DEIMoS's common N-dimensional grid before peak detection or other downstream steps.
when_to_use_negative:
- Input is already a processed feature coordinate table (e.g. example_data_peaks.h5)
  rather than raw mzML; downstream alignment or CCS calibration tools are the right
  choice instead
- Data are not multidimensional (no drift time / ion mobility dimension), since DEIMoS's
  grid-and-multidimensional-algorithm approach offers no advantage over single-dimension
  processing
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Python API and command-line tool that loads mzML(.gz) acquisitions into the
    common m/z, drift time, retention time, intensity grid and provides deimos.load
    for persisted HDF5 grids
  repo: github.com/pnnl/deimos
- name: Python
  role: Runtime environment in which DEIMoS is installed (git clone https://github.com/pnnl/deimos.git)
    and invoked for loading and grid construction
  repo: github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:e2d235637b15cc6fce584830cb36220cd086c8cc8fccd21e9c75ddcc3977f136
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mzML file loading and grid construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for loading mzML(.gz) mass spectrometry files into a common m/z, drift time, retention time, intensity grid as the first DEIMoS processing step, providing the N-dimensional data structure on which all downstream feature detection, alignment, CCS calibration, isotope detection and MS/MS deconvolution operate.

## When to use

Use when raw LC-IMS-MS/MS acquisitions in mzML(.gz) format (e.g. Agilent 6560 Ion Mobility LC/Q-TOF deposits such as MSV000091746) must be converted into DEIMoS's common N-dimensional grid before peak detection or other downstream steps.

## When NOT to use

- Input is already a processed feature coordinate table (e.g. example_data_peaks.h5) rather than raw mzML; downstream alignment or CCS calibration tools are the right choice instead
- Data are not multidimensional (no drift time / ion mobility dimension), since DEIMoS's grid-and-multidimensional-algorithm approach offers no advantage over single-dimension processing

## Inputs

- mzML or gzipped mzML files from LC-IMS-MS/MS acquisitions (Agilent 6560, Bruker, Waters instruments)
- positive and negative ionization mode files processed separately
- instrument deposit accession (e.g. MassIVE MSV000091746)

## Outputs

- Common m/z, drift time, retention time, intensity grid (N-dimensional data structure) for each ionization mode
- HDF5 file of the loaded data (e.g. example_data.h5) ready for deimos.threshold / peakpick / alignment steps

## How to apply

Load the mzML(.gz) files (from e.g. the MassIVE deposit MSV000091746 acquired on the Agilent 6560 Ion Mobility LC/Q-TOF) into DEIMoS's common m/z, drift time, retention time, intensity grid; DEIMoS operates on N-dimensional data largely agnostic to acquisition instrumentation, so the same grid construction applies to Bruker (54 files, 9 study samples in triplicate, 83 GB) and Waters (21 files, 6 study samples in triplicate, 16 GB) acquisitions. Optionally apply a nominal intensity threshold (e.g. threshold=500 per the user guide) right after loading to decrease the signal span of the 3D space prior to peak detection. Process positive and negative ionization mode files separately and persist each loaded grid as HDF5 (e.g. example_data.h5) so that downstream deimos.threshold, deimos.peakpick and alignment functions can consume it. Rationale: a shared grid is what allows DEIMoS's multidimensional algorithms to simultaneously utilize all dimensions and offer greater separation between features, improving detection sensitivity.

## Related tools

- **DEIMoS** (Python API and command-line tool that loads mzML(.gz) acquisitions into the common m/z, drift time, retention time, intensity grid and provides deimos.load for persisted HDF5 grids) — github.com/pnnl/deimos
- **Python** (Runtime environment in which DEIMoS is installed (git clone https://github.com/pnnl/deimos.git) and invoked for loading and grid construction) — github.com/pnnl/deimos

## Examples

```
$ git clone https://github.com/pnnl/deimos.git && python -c "import deimos; ms1 = deimos.load('example_data.h5', key='ms1')"
```

## Evaluation signals

- Loaded grid must contain all four axes used by DEIMoS: m/z, drift time, retention time and intensity
- Positive and negative ionization modes must be loaded and saved as separate HDF5 outputs (mirroring the 912-file / 1.1 TB acquisition handled by the authors)
- Applying the nominal intensity threshold (threshold=500) immediately after loading should reduce the signal span of the 3D space before peak detection
- Downstream deimos.peakpick.partition on the loaded grid must succeed, indicating the grid is correctly formed (peak detection then post-detection thresholding at 1000 follows)

## Limitations

- DEIMoS is 'largely agnostic to acquisition instrumentation' but still requires the input to be provided as mzML(.gz); other vendor-native formats are not mentioned
- Cumulative data volumes can be large (1.1 terabytes for 912 files in the article), so loading all acquisitions at once may be impractical without per-mode/per-file handling

## Evidence

- [results] ms1 = deimos.load(
    "example_data.h5",: "ms1 = deimos.load(
    "example_data.h5","
