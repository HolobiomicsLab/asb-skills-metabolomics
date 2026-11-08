---
name: vendor-raw-file-ingestion-wiff
description: Use when the input data are proprietary vendor raw files (e.g. Sciex
  .WIFF) that must be processed by MS-DIAL 5 from the command line for headless runs,
  benchmarking, or cross-platform reproducibility.
when_to_use_negative:
- You are using the 'Debug/Release vendor unsupported' source distribution — it cannot
  read proprietary formats like WIFF; convert to abf/cdf/mzml or use the official
  release binaries instead.
- Your data are already in open formats (abf, NetCDF cdf, mzML) and you only need
  the standard pipeline — the vendor ingestion path is unnecessary.
- You need interactive inspection or GUI project artifacts — use the Windows GUI (MsdialGuiApp)
  rather than the console light path, which skips GUI chromatogram serialization and
  project saving.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL 5 (MSDIALCUI.exe console)
  role: Reads vendor WIFF raw files headlessly and executes the LC-MS detection/alignment
    workflow; full and alignment-light modes are driven via the method file
  repo: https://github.com/systemsomicslab/MsdialWorkbench
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-54137-w
    title: MS-DIAL 5 multimodal mass spectrometry data mining unveils lipidome complexities
  source_content_hash: sha256:53ff2c01c62ba2cd5a57fcef58e8faa2bac77cbd75811cdb1126b3d1b39d88cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systemsomicslab/MsdialWorkbench
  tools:
  - MS-DIAL 5
  - MS-DIAL 5 (MSDIALCUI.exe console)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# vendor-raw-file-ingestion-wiff

## Summary

Ingests Sciex WIFF vendor raw LC-MS files headlessly through the MS-DIAL 5 console binary (MSDIALCUI.exe), enabling OS-independent processing/benchmarking without the Windows-only GUI.

## When to use

Use when the input data are proprietary vendor raw files (e.g. Sciex .WIFF) that must be processed by MS-DIAL 5 from the command line for headless runs, benchmarking, or cross-platform reproducibility.

## When NOT to use

- You are using the 'Debug/Release vendor unsupported' source distribution — it cannot read proprietary formats like WIFF; convert to abf/cdf/mzml or use the official release binaries instead.
- Your data are already in open formats (abf, NetCDF cdf, mzML) and you only need the standard pipeline — the vendor ingestion path is unnecessary.
- You need interactive inspection or GUI project artifacts — use the Windows GUI (MsdialGuiApp) rather than the console light path, which skips GUI chromatogram serialization and project saving.

## Inputs

- Sciex WIFF vendor raw LC-MS files (e.g. 7 FastLC WIFF demo files)
- analysis_files.csv listing the raw files to process
- method.txt console method file (with 'Msp file path' and 'Text DB file path' set, and optional 'Alignment light mode: True/False')
- MSDIALCUI.exe console binary built in Release/net48 from MsdialWorkbench

## Outputs

- mdalign alignment output file
- mdmsp MSP spectral output file
- mzTab output (with filename-derived 'MTD mzTab-ID' line)
- alignment_light_benchmark.tsv (elapsed seconds and peak private memory MB per mode)
- GUI project file (only in full mode, not in alignment-light mode)

## How to apply

Build the Console binary (MSDIALCUI.exe, Release/net48) from the MsdialWorkbench repository using the dotnet_test.yml build recipe, using an official release configuration because the 'Debug/Release vendor unsupported' build cannot read proprietary manufacturer formats. Locate the FastLC WIFF demo dataset with its analysis_files.csv and method.txt, and point the method's 'Msp file path' to lib/MSMS-Public_all-neg-VS19.msp and 'Text DB file path' to lib/20200121_MsdialTxtDB_Neg_EquiSPLASH_rapid.txt. Run the lcms subcommand (MSDIALCUI.exe lcms -i analysis_files.csv -o <outdir> -m method.txt) headlessly, optionally with 'Alignment light mode: True' in the method file so the LcmsAlignmentLightRunner skips GUI chromatogram serialization, alignment object serialization and '-p' project saving. Measure elapsed time and peak private memory (e.g. with a process memory sampler) when benchmarking, and validate text outputs (mdalign, mdmsp, mzTab) rather than GUI artifacts.

## Related tools

- **MS-DIAL 5 (MSDIALCUI.exe console)** (Reads vendor WIFF raw files headlessly and executes the LC-MS detection/alignment workflow; full and alignment-light modes are driven via the method file) — https://github.com/systemsomicslab/MsdialWorkbench

## Examples

```
MSDIALCUI.exe lcms -i analysis_files.csv -o <out_full> -m method.txt
```

## Evaluation signals

- Alignment-light run: mdalign and mdmsp outputs are byte-identical (SHA256) to full-mode outputs.
- mzTab output is identical to full mode after normalizing the filename-derived 'MTD mzTab-ID' line.
- No GUI project file is generated when 'Alignment light mode: True' is set.
- Benchmark table reproduces the reported comparison (Full: 112.3 s / 854.2 MB; Light: 110.1 s / 837.3 MB) within run-to-run variance.
- Light-mode peak private memory is lower than full-mode peak private memory.

## Limitations

- WIFF is a proprietary Sciex format: the 'Debug/Release vendor unsupported' source configuration only supports abf (Reifycs), cdf (NetCDF), and mzml, so official release binaries are required for WIFF ingestion.
- The MS-DIAL GUI is Windows-only; headless console ingestion is the cross-platform path.
- Alignment-light mode skips GUI chromatogram serialization, alignment object serialization and '-p' project saving, so no GUI project file is produced in that mode.
- Timing/memory benchmark results are subject to run-to-run variance.

## Evidence

- [other] MS-DIAL functions as an operating system (OS)-independent command line tool, although its graphical user interface is available only for Windows OS.: "MS-DIAL functions as an operating system (OS)-independent command line tool, although its graphical user interface is available only for Windows OS."
- [readme] Due to licensing restrictions, this version cannot read proprietary data formats from mass spectrometry manufacturers.: [evidence span withheld — anchor status: external_doc]
- [readme] For the 'Debug/Release vendor unsupported' version, only the following formats are supported: abf(Reifycs), cdf(NetCDF), and mzml.: [evidence span withheld — anchor status: external_doc]
- [results] The source code is publicly accessible from the GitHub repository (https://github.com/systemsomicslab/MsdialWorkbench).: "The source code is publicly accessible from the GitHub repository (https://github.com/systemsomicslab/MsdialWorkbench)"
