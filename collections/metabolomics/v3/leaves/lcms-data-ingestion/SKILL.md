---
name: lcms-data-ingestion
description: Use when a Thermo .raw or .mzML LC-MS data file needs to be loaded into
  lcmsWorld for 3D visualization, especially as the first load of a file that has
  no cached .lcms conversion yet.
when_to_use_negative:
- The data is already in .lcms format — load it directly to start viewing instantly
  instead of re-ingesting the .raw/.mzml.
- A headless or scripted file conversion is required — the README lists 'Headless
  mode for file conversions' only as a to-do item.
- The input is a peptide identification file (.csv or tab-separated text with m/z,
  rt, intensity columns) — these are loaded via the identification/Annotations pathway,
  not the LC-MS ingestion pathway.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: lcmsWorld
  role: The 3D viewer application that performs the 'Load LC-MS' ingestion and automatic
    .lcms conversion of .raw or .mzml files
  repo: https://github.com/PGB-LIV/lcmsWorld
- name: RawFileReader
  role: Thermo library used internally by lcmsWorld to load Thermo .raw files; 64-bit
    only .NET application
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00618
    title: 'lcmsWorld: High-Performance 3D Visualization Software for Mass Spectrometry'
  source_content_hash: sha256:d01621db9ab24ae033215be21fb5dd32c82f6915787e447f537db38cd190232f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PGB-LIV/lcmsWorld
  tools:
  - .NET Framework
  - RawFileReader
  - lcmsWorld
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lcms-data-ingestion

## Summary

Ingests LC-MS data into lcmsWorld by loading Thermo .raw or .mzML files through Thermo RawFileReader (a 64-bit .NET application), which triggers automatic conversion to a .lcms file for fast streaming during visualization.

## When to use

Use when a Thermo .raw or .mzML LC-MS data file needs to be loaded into lcmsWorld for 3D visualization, especially as the first load of a file that has no cached .lcms conversion yet.

## When NOT to use

- The data is already in .lcms format — load it directly to start viewing instantly instead of re-ingesting the .raw/.mzml.
- A headless or scripted file conversion is required — the README lists 'Headless mode for file conversions' only as a to-do item.
- The input is a peptide identification file (.csv or tab-separated text with m/z, rt, intensity columns) — these are loaded via the identification/Annotations pathway, not the LC-MS ingestion pathway.

## Inputs

- Thermo .raw LC-MS data file
- .mzML LC-MS data file
- Example .raw files from http://pgb.liv-ac.uk/lcmsWorld/testfiles

## Outputs

- Automatically generated .lcms file (cached conversion of the loaded .raw/.mzml file, written on the same disk)
- Interactive 3D LC-MS visualization inside lcmsWorld

## How to apply

Confirm .NET Framework 4.7 and OpenGL 3.0+ are available, since RawFileReader is a 64-bit-only .NET application used by lcmsWorld to load .raw files. Launch lcmsWorld.exe, then select the 'File' menu, choose 'Load LC-MS' file, and set the file filter to either .raw or .mzml. On the first load of a file, lcmsWorld automatically converts it and creates a corresponding .lcms file on the same disk; store this on a fast hard disk because data is streamed from the .lcms file during use, and reserve roughly the same amount of hard disk space as the .raw/.mzml being viewed. Alternatively, load the previously generated .lcms file directly to start viewing instantly without reconverting.

## Related tools

- **lcmsWorld** (The 3D viewer application that performs the 'Load LC-MS' ingestion and automatic .lcms conversion of .raw or .mzml files) — https://github.com/PGB-LIV/lcmsWorld
- **RawFileReader** (Thermo library used internally by lcmsWorld to load Thermo .raw files; 64-bit only .NET application)

## Evaluation signals

- A corresponding .lcms file is created next to the source file after the first load, on the same disk.
- The 'Load LC-MS' file dialog accepts the file only when the filter is set to .raw or .mzml.
- The .lcms file size is roughly the same order as the source .raw/.mzml being viewed.
- Subsequent loads of the cached .lcms file start viewing instantly rather than re-running the conversion.
- The 3D view renders with camera controls working (left mouse rotates viewpoint, mouse wheel zooms).

## Limitations

- RawFileReader is a 64-bit only .NET application; .NET Framework 4.7 may need to be installed on Windows.
- Requires OpenGL 3.0+ and at least 1gb RAM; hard disk space about equal to the .raw/.mzml being viewed.
- Little error checking is performed on input files (README to-do: 'Test on various other input files'), and MacOS builds are untested and 'unlikely to work without changes'.
- The Debug build is painfully slow for loading; use a Release build.

## Evidence

- [intro] lcmsWorld supports .raw/.mzml ingestion: "lcmsWorld is a 3d viewer for LC-MS data (it can load ```.mzml``` or Thermo ```.raw``` format files)"
- [methods] Load LC-MS via File menu with .raw/.mzml filter: "select the 'File' menu, 'Load LC-MS' file, and change the file filter to either .raw or .mzml"
- [readme] Automatic conversion to .lcms on first load: "The first time you load a file, lcmsWorld automatically converts this file and creates a corresponding .lcms file."
- [readme] RawFileReader is the .raw loading pathway: "RawFileReader from Thermo is used to load .raw files.  This is a 64-bit only .NET application."
- [readme] .lcms file streamed from fast disk: "For large files, preferably loaded from fast hard disk - the viewable .lcms file is created on the same disk, and 
data is streamed from it during use."
