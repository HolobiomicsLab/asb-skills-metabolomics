---
name: raw-to-mzml-conversion
description: Use when input data are vendor-specific instrument raw files (e.g., .raw
  from Thermo instruments) that must be converted to open-source .mzML before MassQL
  queries or MassQLab workflows can be applied.
when_to_use_negative:
- Input files are already in .mzML (or another open-source) format — conversion is
  unnecessary and MassQLab can process them directly
- Conversion fidelity must be vendor-validated for quantitative downstream analysis
  — the README states MassQLab's built-in raw-to-mzML conversion is experimental
- The analysis targets native vendor binary content (e.g., instrument tuning logs)
  not representable in standard mzML
edam_operation: http://edamontology.org/operation_0325
tools:
- name: MSConvert
  role: ProteoWizard tool used to convert vendor .raw files to mzML with default settings
    before MassQL queries run
  repo: https://proteowizard.sourceforge.io/
- name: MassQLab
  role: Python 3.9 package that consumes the converted mzML files; its massqlab_config.
    exposes convert_raw and msconvertexe for experimental in-pipeline conversion
  repo: https://github.com/JohnsonDylan/MassQLab
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.10132
    title: 'Open‐Source System Suitability: Mass Spectrometry Query Language Lab (MassQLab)'
  source_content_hash: sha256:b494aa65f82bafa5ed47892eafe8ce4f0c00f879522e7de81e1cafb72d4267ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSConvert
  - MassQLab
  - Python
  - matplotlib
  - openpyxl
  - reportlab
  edam_operation: http://edamontology.org/operation_0325
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# raw-to-mzml-conversion

## Summary

Converts vendor-specific mass spectrometry raw files (e.g., Thermo .raw from UHPLC-HRMS acquisition) into the open-source mzML format using MSConvert with default settings, as a prerequisite for downstream MassQL/MassQLab querying.

## When to use

Use when input data are vendor-specific instrument raw files (e.g., .raw from Thermo instruments) that must be converted to open-source .mzML before MassQL queries or MassQLab workflows can be applied.

## When NOT to use

- Input files are already in .mzML (or another open-source) format — conversion is unnecessary and MassQLab can process them directly
- Conversion fidelity must be vendor-validated for quantitative downstream analysis — the README states MassQLab's built-in raw-to-mzML conversion is experimental
- The analysis targets native vendor binary content (e.g., instrument tuning logs) not representable in standard mzML

## Inputs

- Vendor-specific LC-MS/MS raw files (e.g., Thermo .raw from UHPLC-HRMS acquisition of the deuterated acylcarnitine SST mixture)
- Path to msconvert.exe executable (for MassQLab's experimental convert_raw option)

## Outputs

- Open-source .mzML files in the data_directory consumed by MassQLab
- MassQLab_Output results (ms1_raw_df.csv, ms2_raw_df.csv, PDFs) derived from the converted files

## How to apply

Run MSConvert (ProteoWizard) on the raw files using default settings to produce mzML, per the MassQLab methods. Alternatively, set the massqlab_config. fields convert_raw=true and msconvertexe to the msconvert.exe path so MassQLab attempts conversion itself, though the README flags this as experimental and recommends converting with ProteoWizard msconvert beforehand. Point data_directory in massqlab_config. at the folder containing the resulting .mzML files, since MassQLab applies MassQL queries to every file in that directory. Conversion is mandatory: MassQL operates on open-source mzML only, so skipped or failed conversion blocks all downstream query execution.

## Related tools

- **MSConvert** (ProteoWizard tool used to convert vendor .raw files to mzML with default settings before MassQL queries run) — https://proteowizard.sourceforge.io/
- **MassQLab** (Python 3.9 package that consumes the converted mzML files; its massqlab_config. exposes convert_raw and msconvertexe for experimental in-pipeline conversion) — https://github.com/JohnsonDylan/MassQLab

## Examples

```
msconvert MSV000098825/SST_mix.raw --mzML -o mzML_dir
```

## Evaluation signals

- One .mzML file exists per input .raw file in the data_directory, with no conversion failures
- MSConvert was run with default settings, matching the published MassQLab methods
- MassQL queries execute without file-loading errors on the converted mzML (e.g., msql_fileloading.load_data returns populated ms1_df/ms2_df frames)
- massqlab_config. data_directory points at the mzML folder and convert_raw/msconvertexe are consistent with the conversion path actually used

## Limitations

- MassQLab's built-in convert_raw option is explicitly experimental; the README recommends converting with ProteoWizard msconvert beforehand
- Conversion is a hard prerequisite — MassQL cannot query vendor-specific formats directly
- If the massqlab config lacks msconvertexe while convert_raw=true, in-pipeline conversion cannot proceed

## Evidence

- [methods] Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings.: "Raw data files (.raw) were converted to mzML format using MSConvert (https://proteowizard.sourceforge.io/) with default settings."
- [intro] Vendor-specific formats must be converted to open-source before MassQL use.: "One prerequisite is that vendor‐specific file formats must be converted into an open‐source MS file format (i.e., .mzML)."
- [readme] convert_raw is experimental; recommend msconvert beforehand.: "Note: This feature is experimental. It is recommended to convert files using [ProteoWizard msconvert](https://proteowizard.sourceforge.io/) before using MassQLab."
