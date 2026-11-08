---
name: proteowizard-msconvert-conversion
description: Use when raw vendor files (Thermo .raw, Agilent/Bruker .d) must be converted
  to open formats like mzML, mzXML, mz5 or mzMLb for downstream storage-size or access
  benchmarks against MZA.
when_to_use_negative:
- When the goal is to read converted data across languages without conversion — MZA's
  HDF5 structure is the intended interface; this skill is only for producing the reference
  formats.
- When vendor files are already in an open format (e.g. input is mzML) and only re-encoding
  is needed — run the format-specific converter rather than the full vendor-conversion
  path.
- When ion-mobility dimension must be preserved in a lossy-truncated way — the benchmark
  requires lossless conversion with zero-intensity peaks removed, not partial or filtered
  output.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: Toolkit invoked to convert raw vendor files to the open reference formats
    mz5, mzXML, mzML and mzMLb used in the file-size benchmark.
- name: MZA
  role: Converts the same raw files to its HDF5-based structure so that size and access
    characteristics can be compared against the ProteoWizard-produced formats.
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: Underlying binary format of MZA output; relevant only as the storage substrate
    being benchmarked, not as a converter in this skill.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:977d36d52c1e5cb698ebe5b0f174b5c6166b7572eba5f90efd0c826632ad9109
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - ProteoWizard
  - mz5
  - mzXML
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# proteowizard-msconvert-conversion

## Summary

Converts raw MS files from instrument vendors to open interchange formats such as mzML using the ProteoWizard toolkit, enabling storage-efficiency and access-speed comparisons against the MZA HDF5 structure.

## When to use

Use when raw vendor files (Thermo .raw, Agilent/Bruker .d) must be converted to open formats like mzML, mzXML, mz5 or mzMLb for downstream storage-size or access benchmarks against MZA.

## When NOT to use

- When the goal is to read converted data across languages without conversion — MZA's HDF5 structure is the intended interface; this skill is only for producing the reference formats.
- When vendor files are already in an open format (e.g. input is mzML) and only re-encoding is needed — run the format-specific converter rather than the full vendor-conversion path.
- When ion-mobility dimension must be preserved in a lossy-truncated way — the benchmark requires lossless conversion with zero-intensity peaks removed, not partial or filtered output.

## Inputs

- Proprietary vendor raw MS files: Thermo .raw, Agilent MassHunter .d (with or without ion mobility), Bruker TimsTOF .d
- ProteoWizard toolkit installation (msconvert) configured for mzML/mzXML/mz5/mzMLb output
- Target output format specification: mz5, mzXML, mzML, or mzMLb per conversion run

## Outputs

- Converted open-format MS files (mzML, mzXML, mz5, mzMLb) from each raw input
- A per-dataset, per-format file-size table (e.g. file_size_comparison.csv) covering all dataset x format combinations
- Relative ordering of output file sizes across formats for the benchmark report

## How to apply

Take each proprietary vendor file (Thermo .raw, Agilent '.d', Bruker '.d') and run the ProteoWizard conversion toolkit to produce each target open format, one per run. The article benchmarks MZA against mz5, mzXML, mzML and mzMLb, so a separate conversion must be issued for every dataset-format pair. Apply the lossless setting with zero-intensity peaks removed — equivalent to running MZA with no intensity threshold — so that size comparisons are fair. Record the resulting byte size of every output file, e.g. into a file_size_comparison.csv, and verify the relative ordering (mz5 smallest, then mzMLb/MZA, then mzXML/mzML) matches the reported Fig. 2 behaviour. MZA itself is invoked as a stand-alone executable that has no dependencies, whereas ProteoWizard conversions are used purely as the reference point to quantify how MZA's HDF5-based structure trades file size for cross-language accessibility.

## Related tools

- **ProteoWizard** (Toolkit invoked to convert raw vendor files to the open reference formats mz5, mzXML, mzML and mzMLb used in the file-size benchmark.)
- **MZA** (Converts the same raw files to its HDF5-based structure so that size and access characteristics can be compared against the ProteoWizard-produced formats.) — https://github.com/PNNL-m-q/mza
- **HDF5** (Underlying binary format of MZA output; relevant only as the storage substrate being benchmarked, not as a converter in this skill.)

## Examples

```
msconvert --mzML --filter "zeroIntensitiesOff" LCMSMS_Lipids_POS.raw
```

## Evaluation signals

- For every raw-file x target-format combination, the converted output exists and has a nonzero file size.
- Zero-intensity peaks are absent from the converted spectra, matching the MZA default of storing spectra 'omitting zero-intensity values'.
- The recorded file sizes reproduce the reported ordering: mz5 smallest, MZA/mzMLb mid-range, mzXML/mzML largest, most pronounced for the LC-SLIM proteomics file with the largest number of spectra.
- Each output opens and yields the same number of spectra as the source raw file, confirming lossless conversion.
- The compiled benchmark table contains 3 datasets x 5 formats = 15 entries with no missing cells.

## Limitations

- The article notes that 'While improvements in speed and file size are still needed' for MZA, so mzML-based conversions may be faster or smaller in some cases — do not assume MZA wins on size.
- Vendor raw libraries are Windows-bounded; running conversions on Linux requires Wine or Docker containers, adding an environment step.
- Lossless conversion with zero-intensity peaks removed can still produce very large files for high-frame-count SLIM or DIA acquisitions.

## Evidence
