---
name: mass-spectrometry-file-format-conversion
description: Use when raw vendor MS files (e.g., Thermo .raw, Agilent or Bruker .d)
  must be converted to an open format like .mza, mzMLb, mzML or mz5, or when benchmarking
  conversion speed across MS formats.
when_to_use_negative:
- The data are already in an open, cross-platform format (e.g., mzML or HDF5) that
  your downstream Python/R tooling can read directly, so conversion adds no value.
- The analysis target is a downstream quantitative result (e.g., CCS/z visualization
  or feature engineering) rather than the conversion step itself — use the CCSz-demo
  or access scripts instead of re-converting.
- The tool is instrument/application-specific in a way MZA does not support — MZA
  supports Agilent .d, Bruker IM .d, Thermo .raw and mzML inputs only.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MZA
  role: Primary command-line converter from proprietary vendor formats to the HDF5-based
    .mza structure
  repo: https://github.com/PNNL-m-q/mza
- name: ProteoWizard
  role: Alternative converter used in the speed benchmark to generate mzXML, mzML
    and mz5 with optimized parallel conversion
- name: mzMLb
  role: HDF5-based converter used as the comparison baseline that MZA outperformed
    in conversion speed
- name: Docker
  role: mzacontainer image providing a reproducible Unix environment to run mza.exe
    via Wine
- name: Wine
  role: Compatibility layer (tested 6.0.3, with wine-mono 7.3.0) that lets mza.exe
    run on Ubuntu 22.04.1
- name: h5py
  role: Python library for accessing spectra arrays in the converted .mza HDF5 files
- name: rhdf5
  role: R Bioconductor package for accessing spectra arrays in the converted .mza
    HDF5 files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:ae3cc3cb2a94331778388f3766584abe6db0f68baa5aa0a135ac150a771d5f04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - Docker
  - MZA
  - ProteoWizard
  - Wine
  - h5py
  - mz5
  - mzMLb
  - mzXML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-spectrometry-file-format-conversion

## Summary

Convert proprietary vendor MS files (Thermo .raw, Agilent .d, Bruker .d, mzML) into the MZA HDF5-based format — or into mzMLb/mzXML/mzML/mz5 — so multidimensional LC-MS/LC-IM-MS spectra with DIA data can be read cross-platform in Python/R and benchmarked for conversion speed.

## When to use

Use when raw vendor MS files (e.g., Thermo .raw, Agilent or Bruker .d) must be converted to an open format like .mza, mzMLb, mzML or mz5, or when benchmarking conversion speed across MS formats.

## When NOT to use

- The data are already in an open, cross-platform format (e.g., mzML or HDF5) that your downstream Python/R tooling can read directly, so conversion adds no value.
- The analysis target is a downstream quantitative result (e.g., CCS/z visualization or feature engineering) rather than the conversion step itself — use the CCSz-demo or access scripts instead of re-converting.
- The tool is instrument/application-specific in a way MZA does not support — MZA supports Agilent .d, Bruker IM .d, Thermo .raw and mzML inputs only.

## Inputs

- Agilent '.d' MassHunter files (with or without ion mobility)
- Bruker ion mobility '.d' TimsTOF files
- Thermo '.raw' files
- mzML files
- Public raw MS data (e.g., MassIVE dataset MSV000090269)

## Outputs

- .mza files (HDF5-based: Metadata table, Arrays_intensity, Arrays_mz, and for IM data Arrays_mzbin + Full_mz_array)
- mzMLb, mzXML, mzML and mz5 files (via mzMLb / ProteoWizard)
- conversion_benchmark_times.csv (per raw-file/format wall-clock times)

## How to apply

Download mza.exe from the PNNL-m-q/mza releases and run it directly on Windows or under Wine (6.0.3 + wine-mono 7.3.0) / the mzacontainer Docker image on Unix. Convert a single file with `mza -file <raw> -intensityThreshold 20` or batch-convert a directory by supplying `-extension .d`, `.raw` or `.mzML`; the intensityThreshold parameter sets the minimum intensity below which signals are omitted from the output .mza file. The resulting file is a simple HDF5 structure: one metadata table plus Arrays_intensity and Arrays_mz 1D arrays stored per spectrum and named by Scan (with Arrays_mzbin and Full_mz_array for IM data). For cross-format speed benchmarks, convert the same three raw files with mzMLb and with ProteoWizard's optimized parallel mode to mzXML, mzML and mz5, timing each conversion identically, and compile per-file/per-format wall-clock times into a CSV.

## Related tools

- **MZA** (Primary command-line converter from proprietary vendor formats to the HDF5-based .mza structure) — https://github.com/PNNL-m-q/mza
- **ProteoWizard** (Alternative converter used in the speed benchmark to generate mzXML, mzML and mz5 with optimized parallel conversion)
- **mzMLb** (HDF5-based converter used as the comparison baseline that MZA outperformed in conversion speed)
- **Docker** (mzacontainer image providing a reproducible Unix environment to run mza.exe via Wine)
- **Wine** (Compatibility layer (tested 6.0.3, with wine-mono 7.3.0) that lets mza.exe run on Ubuntu 22.04.1)
- **h5py** (Python library for accessing spectra arrays in the converted .mza HDF5 files)
- **rhdf5** (R Bioconductor package for accessing spectra arrays in the converted .mza HDF5 files)

## Examples

```
mza -file test_data/LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Output .mza file opens with a generic HDF5 library and contains a Metadata table plus Arrays_intensity and Arrays_mz groups whose datasets are named by the Scan value (e.g., Arrays_intensity/630, Arrays_mz/630 for Scan 630).
- For IM inputs, Arrays_mzbin indexes point into Full_mz_array, and IonMobilityBin = 0 marks the total frame (IM-dimension-ignored) spectrum.
- Setting -intensityThreshold (e.g., 20) excludes signals below the threshold, so array sizes shrink compared with a threshold-free conversion of the same file.
- In the cross-format benchmark, the qualitative ordering of mean conversion times should match: MZA faster than mzMLb, but slower than ProteoWizard's optimized parallel mzXML/mzML/mz5 conversions.
- Conversion succeeds for each supported input type (Agilent .d, Bruker IM .d, Thermo .raw, mzML) both natively on Windows and via Wine/Docker on Unix.

## Limitations

- Improvements in speed and file size are still needed for MZA; no quantitative speed or file-size benchmarks are reported in the discussion section.
- The Windows executable must run under Wine (6.0.3 + wine-mono 7.3.0) or the mzacontainer Docker image on Unix-like systems, adding deployment overhead.
- Files with very many spectra require partitioned HDF5 groups (MzaPath index) because storing many datasets in one group slows data reading; no changelog or version number is stated for the MZA repository.

## Evidence
