---
name: mass-spectrometry-format-conversion
description: Use when the input is an LC-MS data file in .mzML or Thermo .raw format
  and the goal is to parse it into flat per-spectrum records (spectrum_id, rt, mz,
  intensity) for downstream viewing, tabulation, or reuse without a GUI.
when_to_use_negative:
- Input is already a converted .lcms file or an already-flattened spectrum table —
  lcmsWorld loads .lcms directly to start viewing instantly, no re-parsing needed
- Input is an identification file (.csv or tab-separated with m/z, rt, intensity,
  peptide columns) — these are annotation/identification files loaded via the Annotations
  pathway, not LC-MS data conversions
- The goal is interactive 3D viewing via the GUI — use lcmsWorld's 'Load LC-MS' directly
  rather than reimplementing the parsing pathway
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: lcmsWorld
  role: Reference implementation of the .mzML/.raw ingestion and conversion-to-.lcms
    pathway; built from source with cmake on Linux/MacOS or via MS Visual Studio 2019
    on Windows
  repo: https://github.com/PGB-LIV/lcmsWorld
- name: RawFileReader
  role: Thermo library used by lcmsWorld to load .raw files during the conversion
    pathway
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00618
    title: 'lcmsWorld: High-Performance 3D Visualization Software for Mass Spectrometry'
  source_content_hash: sha256:76d7f96e7fdcfa22f4f3b45e12ae2dd5ef924299de06fe82d4d83cfe96d4970f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PGB-LIV/lcmsWorld
  tools:
  - RawFileReader
  - lcmsWorld
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mass-spectrometry-format-conversion

## Summary

Reconstructs the format-conversion pathway lcmsWorld performs when ingesting LC-MS data: parsing .mzML (XML-encoded spectrum elements) or Thermo .raw (via RawFileReader) into per-spectrum records of spectrum ID, retention time, and binary-encoded m/z/intensity peak arrays, converting them into a tabular/.lcms-like representation.

## When to use

Use when the input is an LC-MS data file in .mzML or Thermo .raw format and the goal is to parse it into flat per-spectrum records (spectrum_id, rt, mz, intensity) for downstream viewing, tabulation, or reuse without a GUI.

## When NOT to use

- Input is already a converted .lcms file or an already-flattened spectrum table — lcmsWorld loads .lcms directly to start viewing instantly, no re-parsing needed
- Input is an identification file (.csv or tab-separated with m/z, rt, intensity, peptide columns) — these are annotation/identification files loaded via the Annotations pathway, not LC-MS data conversions
- The goal is interactive 3D viewing via the GUI — use lcmsWorld's 'Load LC-MS' directly rather than reimplementing the parsing pathway

## Inputs

- LC-MS data file in .mzML format (XML-encoded spectrum elements with binary m/z and intensity arrays)
- LC-MS data file in Thermo .raw format (handled via RawFileReader, 64-bit .NET)

## Outputs

- Tab-separated table mzml_spectra.tsv with one row per spectrum and columns spectrum_id, rt, mz, intensity (mz and intensity as delimited lists of paired values)
- Optionally, a converted .lcms file corresponding to the input, as lcmsWorld creates on first load

## How to apply

Mirror the conversion step lcmsWorld runs the first time a file is loaded via the 'Load LC-MS' action. For .mzML input, read the XML-encoded spectrum elements sequentially, extracting each spectrum identifier, the scan retention time, and the binary-encoded m/z and intensity peak arrays (decoding base64 peak lists into paired m/z–intensity values). For Thermo .raw input, the equivalent pathway goes through Thermo's RawFileReader, a 64-bit .NET component requiring .NET Framework 4.7 on Windows. Flatten the parsed records to one row per spectrum with columns spectrum_id, rt, mz, and intensity (mz and intensity as delimited lists of paired values), written to a tab-separated file such as mzml_spectra.tsv. Size the output storage to roughly the same disk footprint as the input .mzml/.raw file, consistent with how lcmsWorld creates the .lcms file on the same disk and streams from it. Validate by confirming the output exists, carries the required fields, and that parsing completes over the whole input file without errors.

## Related tools

- **lcmsWorld** (Reference implementation of the .mzML/.raw ingestion and conversion-to-.lcms pathway; built from source with cmake on Linux/MacOS or via MS Visual Studio 2019 on Windows) — https://github.com/PGB-LIV/lcmsWorld
- **RawFileReader** (Thermo library used by lcmsWorld to load .raw files during the conversion pathway)

## Examples

```
python parse_mzml.py --input sample.mzML --output mzml_spectra.tsv --fields spectrum_id,rt,mz,intensity
```

## Evaluation signals

- Output table mzml_spectra.tsv exists and contains the required fields spectrum_id, rt, mz, and intensity
- Parsing script runs to completion over the whole input .mzML file without parsing errors
- One output row per spectrum element in the input XML; mz and intensity columns contain paired, delimited value lists of equal length
- Disk footprint of the converted output is roughly the same as the input .raw/.mzml file, matching lcmsWorld's documented requirement

## Limitations

- The README notes 'little error checking is performed' and conversion robustness across vendor files is untested ('Test on various other input files' is listed as to-do)
- A headless mode for file conversions is listed as to-do, so conversion is normally triggered through the GUI 'Load LC-MS' action
- .raw loading depends on RawFileReader, a 64-bit only .NET application requiring .NET Framework 4.7 on Windows
- For large files, the converted file should be created on a fast hard disk since data is streamed from it during use; MacOS builds are 'unlikely to work without changes'

## Evidence

- [intro] lcmsWorld is a 3d viewer for LC-MS data that can load .mzml or Thermo .raw format files: "lcmsWorld is a 3d viewer for LC-MS data (it can load ```.mzml``` or Thermo ```.raw``` format files)"
- [methods] First-load conversion creates .lcms file: "The first time you load a file, lcmsWorld automatically converts this file and creates a corresponding .lcms file"
- [methods] Load LC-MS file filter selection: "select the 'File' menu, 'Load LC-MS' file, and change the file filter to either .raw or .mzml"
- [readme] Disk footprint and streaming requirement: "Roughly the same amount of hard disk space as the .raw, .mzml being viewed. For large files, preferably loaded from fast hard disk"
- [readme] RawFileReader dependency for .raw loading: "RawFileReader from Thermo is used to load .raw files.  This is a 64-bit only .NET application."
