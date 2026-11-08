---
name: rawfilereader-api-invocation
description: Use when LC-MS input data is in Thermo vendor .raw format and needs to
  be ingested (or converted to .lcms) for visualization or scan-level extraction,
  e.g. enumerating scan number, retention time, and base peak intensity per scan.
when_to_use_negative:
- The data is already in an open format such as .mzML or an already-converted .lcms
  file that can be loaded directly for instant viewing
- A non-Windows, non-.NET environment is required and building a custom conversion
  path is out of scope (macOS support is untested and 'unlikely to work without changes')
- The goal is identification-level annotation input; identification files (.csv/tab-separated
  with m/z, rt, intensity columns) are loaded separately via the File menu, not via
  RawFileReader
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RawFileReader
  role: Thermo's component used to load .raw vendor files; 64-bit only .NET application
    invoked during ingestion
- name: lcmsWorld
  role: 3D LC-MS viewer whose .raw load pathway (File > Load LC-MS with .raw filter)
    exercises RawFileReader and triggers automatic .lcms conversion
  repo: https://github.com/PGB-LIV/lcmsWorld
- name: .NET Framework 4.7
  role: Runtime prerequisite that may need to be installed on Windows for RawFileReader
    to run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00618
    title: 'lcmsWorld: High-Performance 3D Visualization Software for Mass Spectrometry'
  source_content_hash: sha256:99d368e6ec08ae29be5f75f539879a6bdde4b5076936ee9b8306b21d01badb03
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PGB-LIV/lcmsWorld
  tools:
  - .NET Framework
  - .NET Framework 4.7
  - RawFileReader
  - lcmsWorld
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rawfilereader-api-invocation

## Summary

Load Thermo .raw LC-MS data files through Thermo's RawFileReader, the 64-bit .NET 4.7 ingestion component used by lcmsWorld to read vendor-format mass spectrometry files alongside .mzML.

## When to use

Use when LC-MS input data is in Thermo vendor .raw format and needs to be ingested (or converted to .lcms) for visualization or scan-level extraction, e.g. enumerating scan number, retention time, and base peak intensity per scan.

## When NOT to use

- The data is already in an open format such as .mzML or an already-converted .lcms file that can be loaded directly for instant viewing
- A non-Windows, non-.NET environment is required and building a custom conversion path is out of scope (macOS support is untested and 'unlikely to work without changes')
- The goal is identification-level annotation input; identification files (.csv/tab-separated with m/z, rt, intensity columns) are loaded separately via the File menu, not via RawFileReader

## Inputs

- Thermo .raw LC-MS data file (vendor format)
- .mzML LC-MS data file (alternative supported input)
- Windows environment with .NET Framework 4.7 installed

## Outputs

- In-memory LC-MS scan data (scan number, retention time, base peak intensity per scan)
- Corresponding .lcms file auto-created next to the source .raw/.mzML on first load
- Structured scan-record table (e.g. scan_records.tsv, one row per scan)

## How to apply

Ensure the Windows environment has .NET Framework 4.7 installed, since RawFileReader is a 64-bit only .NET application. Obtain Thermo .raw files (example files are published at the lcmsWorld testfiles URL) and load them either programmatically via RawFileReader or through lcmsWorld's File > Load LC-MS menu with the file filter set to .raw. On first load, lcmsWorld automatically converts the .raw file and writes a corresponding .lcms file on the same (preferably fast) disk, which is then streamed during viewing. For scan-record extraction, iterate over the MS scan records via RawFileReader and collect scan number, retention time, and base peak intensity per scan into a structured table. Note that a headless conversion mode is listed as a to-do, so non-interactive conversion currently relies on the GUI load path or custom .NET code against RawFileReader.

## Related tools

- **RawFileReader** (Thermo's component used to load .raw vendor files; 64-bit only .NET application invoked during ingestion)
- **lcmsWorld** (3D LC-MS viewer whose .raw load pathway (File > Load LC-MS with .raw filter) exercises RawFileReader and triggers automatic .lcms conversion) — https://github.com/PGB-LIV/lcmsWorld
- **.NET Framework 4.7** (Runtime prerequisite that may need to be installed on Windows for RawFileReader to run)

## Examples

```
cmake . && cmake --build . --config Release && ./lcmsWorld  # then File > Load LC-MS, set filter to .raw
```

## Evaluation signals

- The .raw file loads without a .NET/64-bit runtime error, confirming .NET Framework 4.7 and the 64-bit RawFileReader dependency are satisfied
- A corresponding .lcms file appears on the same disk as the source .raw after the first load
- Every scan in the source file yields exactly one record with non-null scan number, retention time, and base peak intensity (one row per scan in scan_records.tsv)
- Extracted retention-time values are monotonically increasing across scan numbers, consistent with a chronologically ordered LC-MS run

## Limitations

- RawFileReader is a 64-bit only .NET application; a .NET Framework 4.7 install may be required on Windows
- Headless mode for file conversions is a to-do item, so conversion currently requires the GUI or custom .NET code
- Debug builds are 'painfully slow for loading'; use Release builds
- Large .raw/.mzML files need roughly equal free disk space on a fast disk since the .lcms file is streamed from the same disk during use
- Little error checking is performed (per the README's to-do list), so malformed input files may fail unpredictably

## Evidence
