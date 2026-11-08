---
name: conversion-benchmark-timing-measurement
description: Use when you need to measure and compare how fast different MS data conversion
  tools (MZA vs mzMLb vs ProteoWizard) convert the same vendor raw files (.d, .raw)
  to open formats.
when_to_use_negative:
- When the goal is data access or downstream analysis of .mza files rather than benchmarking
  conversion speed (use MZA's HDF5 data-access scripts instead).
- When absolute cross-platform speed numbers are required — MZA's Docker/Wine execution
  path adds overhead and the paper reports no quantitative benchmarks, only a qualitative
  ordering.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: Converts vendor raw MS files (.d, .raw) to the HDF5-based .mza format; timed
    against other converters
  repo: https://github.com/PNNL-m-q/mza
- name: ProteoWizard
  role: Converts raw files to mzXML, mzML and mz5 using optimized parallel conversion
    mode; timed as comparison baseline
- name: Docker
  role: mzacontainer image used to run MZA via Wine on Unix-like systems during benchmarking
- name: Wine
  role: Compatibility layer (tested version 6.0.3 with wine-mono 7.3.0) enabling mza.exe
    runs on Unix for timing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:f5df50522268043d756af84cd944d136f637d630bcd120560b4a656b51dd7797
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - Docker
  - MZA
  - ProteoWizard
  - Wine
  - mz5
  - mzMLb
  - mzXML
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# conversion-benchmark-timing-measurement

## Summary

Benchmarking methodology for comparing wall-clock conversion speed of vendor raw MS files across open formats (.mza, mzMLb, mzML, mzXML, mz5), using MZA and ProteoWizard. It yields a qualitative ordering of converters rather than absolute speed claims.

## When to use

Use when you need to measure and compare how fast different MS data conversion tools (MZA vs mzMLb vs ProteoWizard) convert the same vendor raw files (.d, .raw) to open formats.

## When NOT to use

- When the goal is data access or downstream analysis of .mza files rather than benchmarking conversion speed (use MZA's HDF5 data-access scripts instead).
- When absolute cross-platform speed numbers are required — MZA's Docker/Wine execution path adds overhead and the paper reports no quantitative benchmarks, only a qualitative ordering.

## Inputs

- Vendor raw MS files (Agilent .d, Thermo .raw) from public MassIVE dataset MSV000090269
- MZA executable (mza.exe) with -file and optional -extension/-intensityThreshold arguments
- mzMLb converter
- ProteoWizard converter (msconvert) with parallel conversion mode

## Outputs

- conversion_benchmark_times.csv with one row per raw-file/format combination and per-file wall-clock conversion times
- Qualitative ordering of mean conversion times across formats (.mza, mzMLb, mzML, mzXML, mz5)

## How to apply

Download the identical vendor raw files for all converters — here the three example files from the public MassIVE dataset MSV000090269 (Agilent .d and Thermo .raw types). Convert each raw file once per format, timing wall-clock duration identically: with MZA (e.g. 'mza -file <raw> -intensityThreshold 20' natively on Windows, or via the mzacontainer Docker image / Wine 6.0.3 with wine-mono 7.3.0 on Unix), with the mzMLb converter, and with ProteoWizard to mzXML, mzML and mz5 using its optimized parallel conversion mode. Compile per-file, per-format times into a single CSV (conversion_benchmark_times.csv, one row per raw-file/format combination). Judge correctness qualitatively: the measured ordering of mean conversion times should match the reported result — MZA consistently faster than mzMLb but slower than optimized parallel ProteoWizard conversions, consistent with the paper's note that 'improvements in speed and file size are still needed'.

## Related tools

- **MZA** (Converts vendor raw MS files (.d, .raw) to the HDF5-based .mza format; timed against other converters) — https://github.com/PNNL-m-q/mza
- **ProteoWizard** (Converts raw files to mzXML, mzML and mz5 using optimized parallel conversion mode; timed as comparison baseline)
- **Docker** (mzacontainer image used to run MZA via Wine on Unix-like systems during benchmarking)
- **Wine** (Compatibility layer (tested version 6.0.3 with wine-mono 7.3.0) enabling mza.exe runs on Unix for timing)

## Examples

```
mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Measured ordering of mean conversion times matches the reported result: MZA faster than mzMLb, slower than optimized parallel ProteoWizard (mzXML, mzML, mz5)
- conversion_benchmark_times.csv contains one row per raw-file/format combination with no missing timing entries
- Each conversion timed with identical wall-clock methodology across all formats and files
- Same three raw MS files (from MSV000090269) used for every converter, so comparisons are paired
- MZA conversions succeed on both Windows-native and Docker/Wine paths with the same intensityThreshold setting

## Limitations

- The paper reports no quantitative speed or file-size benchmarks — only the qualitative statement that 'improvements in speed and file size are still needed'
- MZA runs on Unix only through Docker or Wine (tested Ubuntu 22.04.1, Wine 6.0.3, wine-mono 7.3.0), which may confound cross-platform timing comparisons
- No version number stated for the MZA tool and no changelog available, hindering reproducibility of the benchmark
- MZA was consistently slower than ProteoWizard's optimized parallel processing for mzXML, mzML and mz5

## Evidence

- [other] MZA was consistently faster than mzMLb for conversion, but slower than the optimized parallel processing available in ProteoWizard: [evidence span withheld — anchor status: not_found]
- [discussion] While improvements in speed and file size are still needed, MZA provides a practical interface: "While improvements in speed and file size are still needed, MZA provides a practical interface between researchers and MS instrument vendors"
- [methods] mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20: "mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20"
- [readme] MZA is a stand-alone and self-contained command-line executable which converts multidimensional mass spectrometry (MS) data from files in proprietary vendor formats: "MZA is a stand-alone and self-contained command-line executable which converts multidimensional mass spectrometry (MS) data from files in proprietary vendor formats to the MZA simple data structure"
- [readme] docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw: "docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw"
