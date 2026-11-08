---
name: mass-spectrometry-vendor-file-conversion
description: Use when raw vendor MS files (e.g. SCIEX TripleTOF, Thermo Q-Exactive,
  Agilent) must be converted to an open format such as Aird, mzML, mgf, or mz5 before
  DDA or DIA/SWATH analysis, or when comparing file sizes across formats.
when_to_use_negative:
- The input is already an open format such as mzML, mzXML, mgf, or mz5 — conversion
  is unnecessary and may lose vendor metadata.
- Acquisition mode is ScanningSWATH or Ion Mobility that the current Aird version
  does not yet support (under development); convert to mzML instead.
- The analysis requires PSI-MS standard compliance or lossless vendor fidelity — Aird
  uses controllable (lossy) precision on the m/z dimension, so MSNumpress or mzML
  is more appropriate.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AirdPro
  role: GUI conversion client that reads vendor files via ProteoWizard SDK and writes
    the paired .aird/.json Aird format with automatic acquisition-method detection
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MSConvert
  role: ProteoWizard command-line tool used to convert the same raw vendor files into
    mzML, mgf, and mz5 reference formats for comparison
- name: Aird-SDK
  role: Java/C#/Python developer SDK for programmatically reading the converted .aird/.json
    files in downstream analysis
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:9d8cdafeb705a4602d2bb44394a1500da9741ca42880788e988695f95554ffe3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mass-spectrometry-vendor-file-conversion

## Summary

Convert raw vendor mass spectrometry files (SCIEX WIFF, Thermo, Agilent) into open, analysis-ready formats — Aird via AirdPro and mzML/mgf/mz5 via MSConvert — for cross-platform compatibility, storage-efficiency comparison, and downstream computational workflows.

## When to use

Use when raw vendor MS files (e.g. SCIEX TripleTOF, Thermo Q-Exactive, Agilent) must be converted to an open format such as Aird, mzML, mgf, or mz5 before DDA or DIA/SWATH analysis, or when comparing file sizes across formats.

## When NOT to use

- The input is already an open format such as mzML, mzXML, mgf, or mz5 — conversion is unnecessary and may lose vendor metadata.
- Acquisition mode is ScanningSWATH or Ion Mobility that the current Aird version does not yet support (under development); convert to mzML instead.
- The analysis requires PSI-MS standard compliance or lossless vendor fidelity — Aird uses controllable (lossy) precision on the m/z dimension, so MSNumpress or mzML is more appropriate.

## Inputs

- Raw vendor mass spectrometry files: SCIEX .wiff/.wiff2, Thermo RAW, Agilent .d folder format
- ProteoWizard pwiz_bindings_cli.dll (used by AirdPro internally to read vendor data)
- Optional conversion parameters: targeted m/z precision (default 0.0001), compressor choice (ZDPD, Stack-ZDPD, ZDVB), acquisition-mode detection

## Outputs

- Aird data file (.aird) with paired JSON index file (.json), same base name and same directory
- Reference open formats produced by MSConvert: mzML, mgf, mz5 (for comparison or downstream analysis)
- File-size comparison records (bytes per format per dataset) supporting compression-rate evaluation

## How to apply

For Aird conversion, run AirdPro (the C# GUI client built on ProteoWizard's pwiz_bindings_cli.dll), which reads the vendor file via the ProteoWizard SDK, auto-detects the acquisition method (DDA, DIA/SWATH, PRM, DIA_PASEF, DDA_PASEF, MRM), and then compresses the m/z and intensity arrays according to user-specified targeted precision (default mzPrecision 0.0001, i.e. 4-decimal m/z) using ComboComp/Stack-ZDPD compressors. AirdPro reorders spectra per acquisition mode (e.g. MS1-MS2 grouping for DDA, precursor m/z window ranges for DIA/SWATH) to build correlation indexes for fast random access, and always writes a paired .json index file and .aird data file with identical base names in the same directory. For reference-format conversion, use MSConvert from ProteoWizard to produce mzML, mgf, or mz5 from the same raw file, then record the byte size of each output together with the vendor file to quantify the compression achieved (Aird averaged ~2.0 GB in DDA mode, ~54% of RAW size, and ~24.6 GB, ~77% of RAW, in the DIA dataset of the seven-accession comparison). Decision point: keep zero-intensity pairs dropped and optionally enable the log10 intensity compressor only if downstream tools accept log-scaled intensity; verify the .json index is readable as plain text before analysis.

## Related tools

- **AirdPro** (GUI conversion client that reads vendor files via ProteoWizard SDK and writes the paired .aird/.json Aird format with automatic acquisition-method detection) — https://github.com/CSi-Studio/AirdPro/releases/
- **MSConvert** (ProteoWizard command-line tool used to convert the same raw vendor files into mzML, mgf, and mz5 reference formats for comparison)
- **Aird-SDK** (Java/C#/Python developer SDK for programmatically reading the converted .aird/.json files in downstream analysis) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
AirdPro.exe --input raw.wiff --output converted/ ; msconvert raw.RAW --mzML --mgf --mz5 -o converted/
```

## Evaluation signals

- After AirdPro conversion, the output directory contains both a .json index file and a .aird data file with identical base names — missing either signals a failed conversion.
- The .json index file loads as plain text and contains valid AirdInfo fields (version, compressors, indexList, totalCount, msType) matching the source vendor metadata.
- Recorded byte sizes of .aird and mzML outputs relative to the vendor RAW file fall in the reported range: Aird roughly 10–20% smaller than vendor/mzML on average in DDA mode (~2.0 GB average, ~54% of RAW).
- Spectrum count in the Aird index (totalCount) matches the number of spectra readable from the vendor file via ProteoWizard — no lost scans.
- mzML output from MSConvert passes standard mzML schema validation and can be re-read by downstream tools (e.g. mzIdentML-aware pipelines).

## Limitations

- Aird uses controllable-precision (integer, delta-coded) m/z values — lossy relative to the vendor file; users requiring lossless m/z must choose a different format.
- At the time of the article, Aird supported only DDA, DIA/SWATH, and PRM acquisition modes; ScanningSWATH and Ion Mobility were under development (the README now adds DIA_PASEF, DDA_PASEF, MRM).
- Zero-intensity points are dropped during conversion (Ignore Zero Intensity Point filter), which is fine for most computational scenarios but discards data that some workflows may expect.
- The paper reports no benchmark scripts to reproduce its ZDPD-vs-Zlib compression/decoding claims; conversion speed and compression depend on the AirdPro release used.

## Evidence

- [methods] AirdPro uses the integrated SDK provided by ProteoWizard to read vendor file data for the first step of conversion: "AirdPro uses the integrated SDK provided by ProteoWizard [9] to read vendor file data for the first step of conversion"
- [readme] Aird file structure requires paired index and data files: "Aird Index File Suffix: .json <br/>
Aird Data File Suffix: .aird <br/>
Aird Index File and Aird Data File show be stored in the same directory with the same file."
- [abstract] MSConvert produces reference formats for comparison: "Then we convert the raw data into mzML, mgf, and mz5 file formats by MSConvert and compare them with Aird format."
- [readme] Automatic acquisition-method detection is a feature of AirdPro: "Automatic identification for Acquisition methods"
- [methods] Zero-intensity pairs are dropped during conversion: "Ignore Zero Intensity Point. Aird would not store the zero-intensity pairs as most computation scenarios will not use these data."
- [results] Supported acquisition methods currently include DIA, DDA, PRM; ScanningSWATH and Ion Mobility are under development: "ScanningSWATH and Ion Mobility are the formats under development."
- [other] Default m/z precision is 0.0001: [evidence span withheld — anchor status: not_found]
- [results] Aird file size is a fraction of the RAW file size in the reported datasets: [evidence span withheld — anchor status: not_found]
