---
name: mzml-spectrum-xml-parsing
description: Use when the input is a .mzML LC-MS data file (or a Thermo .raw file
  needing the analogous pathway) and the goal is to recover per-spectrum records —
  spectrum_id, rt, m/z and intensity peak lists — into a flat tabular form.
when_to_use_negative:
- Input is already a converted/viewable form such as lcmsWorld's .lcms file, or a
  feature/identification table (.csv or tab-separated with m/z, rt, intensity columns)
  rather than raw spectral XML
- The task is 3D visualization of the LC-MS data in lcmsWorld itself rather than extracting
  spectrum records into a tabular structure
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: lcmsWorld
  role: 'Reference implementation of the .mzML loading pathway: reads .mzml (or Thermo
    .raw) via the ''Load LC-MS'' action with a .mzml filter and converts each file
    on first load; its logic defines how spectrum elements are parsed'
  repo: https://github.com/PGB-LIV/lcmsWorld
- name: RawFileReader
  role: Thermo's component used by lcmsWorld only for the .raw (not .mzml) input pathway
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00618
    title: 'lcmsWorld: High-Performance 3D Visualization Software for Mass Spectrometry'
  source_content_hash: sha256:f7048e045d0817c68f7406e17792e6324557e95eac0d8db5f8eee4c348585295
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PGB-LIV/lcmsWorld
  tools:
  - RawFileReader
  - lcmsWorld
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mzml-spectrum-xml-parsing

## Summary

Implements the .mzML parsing pathway that lcmsWorld performs when loading an LC-MS file: sequentially read XML-encoded spectrum elements and extract each spectrum's identifier, scan retention time, and binary-encoded m/z and intensity peak arrays for reuse outside the viewer.

## When to use

Use when the input is a .mzML LC-MS data file (or a Thermo .raw file needing the analogous pathway) and the goal is to recover per-spectrum records — spectrum_id, rt, m/z and intensity peak lists — into a flat tabular form.

## When NOT to use

- Input is already a converted/viewable form such as lcmsWorld's .lcms file, or a feature/identification table (.csv or tab-separated with m/z, rt, intensity columns) rather than raw spectral XML
- The task is 3D visualization of the LC-MS data in lcmsWorld itself rather than extracting spectrum records into a tabular structure

## Inputs

- LC-MS data file in .mzml format (XML-encoded spectra with binary-encoded m/z and intensity peak arrays)
- Optionally, Thermo .raw format files handled via the analogous lcmsWorld/RawFileReader pathway

## Outputs

- Flat spectrum table written to mzml_spectra.tsv with one row per spectrum and columns spectrum_id, rt, mz, intensity (m/z and intensity as delimited lists of paired values)

## How to apply

Read the XML-encoded spectrum elements sequentially from the .mzML file, mirroring the conversion lcmsWorld performs the first time a user selects 'Load LC-MS' with the .mzml filter. From each spectrum element, extract the spectrum identifier, the scan retention time, and the binary-encoded m/z and intensity peak arrays. Flatten the parsed records into a tabular structure with one row per spectrum and columns for spectrum_id, rt, mz, and intensity (mz and intensity as delimited lists of paired values). Write the records to mzml_spectra.tsv, sizing the work to roughly the same disk footprint as the input .mzML being viewed, consistent with lcmsWorld's requirement that the converted file occupy a similar amount of disk space on a fast disk.

## Related tools

- **lcmsWorld** (Reference implementation of the .mzML loading pathway: reads .mzml (or Thermo .raw) via the 'Load LC-MS' action with a .mzml filter and converts each file on first load; its logic defines how spectrum elements are parsed) — https://github.com/PGB-LIV/lcmsWorld
- **RawFileReader** (Thermo's component used by lcmsWorld only for the .raw (not .mzml) input pathway)

## Examples

```
python parse_mzml_to_tsv.py --input sample.mzml --output mzml_spectra.tsv
```

## Evaluation signals

- mzml_spectra.tsv exists after the script completes
- Output table contains the required fields: spectrum_id, rt, mz, intensity
- Script runs to completion over the whole input .mzML file without parsing errors
- Number of rows equals the number of spectrum elements in the source .mzML
- Each mz/intensity pair list is length-matched (paired m/z and intensity values per spectrum)

## Limitations

- The README notes 'little error checking is performed' and better input-file testing is a listed to-do, so unusual or malformed .mzML files may not parse cleanly
- Conversion/inspection is interactive and disk-bound: lcmsWorld creates the converted file on the same disk and streams from it, needing roughly the same disk space as the .mzml being viewed and preferably a fast disk for large files
- A headless conversion mode is listed only as a to-do, so bulk scripted conversion is not an off-the-shelf viewer feature
- .raw loading depends on Thermo RawFileReader, a 64-bit only .NET application that may require .NET Framework 4.7 on Windows

## Evidence

- [intro] lcmsWorld is a 3d viewer for LC-MS data that can load .mzml or Thermo .raw format files: "lcmsWorld is a 3d viewer for LC-MS data (it can load ```.mzml``` or Thermo ```.raw``` format files)"
- [readme] Load LC-MS file via .mzml filter and first-load conversion: "you should select the 'File' menu, 'Load LC-MS' file, and change the file filter to either .raw or .mzml. The first time you load a file, lcmsWorld automatically converts this file"
- [readme] Disk footprint requirement for converted data: "Roughly the same amount of hard disk space as the .raw, .mzml being viewed."
