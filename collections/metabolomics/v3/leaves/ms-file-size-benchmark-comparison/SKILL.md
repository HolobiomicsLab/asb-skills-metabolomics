---
name: ms-file-size-benchmark-comparison
description: Use when choosing an open mass spectrometry file format for storing converted
  vendor raw data and you need an empirical file-size comparison across formats like
  MZA, mz5, mzXML, mzML and mzMLb.
when_to_use_negative:
- You need speed/parse-time benchmarks rather than storage footprint - the article
  reports improvements in speed are still needed for MZA and does not provide runtime
  benchmarks for this comparison.
- Your comparison requires identical conversion parameters across formats that you
  cannot reproduce (e.g. no ProteoWizard version control), since the size ordering
  depends on equivalent lossless settings.
- You are converting only a single small file; the benchmark's discriminative signal
  comes from datasets with large numbers of spectra such as the LC-SLIM-MS file.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: Converts vendor raw MS files (.d, .raw) to the HDF5-based MZA file structure,
    omitting zero-intensity peaks, to be size-benchmarked against other formats.
  repo: https://github.com/PNNL-m-q/mza
- name: ProteoWizard
  role: Generates the comparison formats mz5, mzXML, mzML and mzMLb from the same
    vendor raw files under equivalent lossless conversion settings.
- name: HDF5
  role: Underlying binary hierarchical format that stores MZA's metadata table and
    jagged arrays_intensity/arrays_mz (+arrays_mzbin) datasets.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:c8ccee449cf3769e3974e8576e318b481cb20afdacd1c0c81734de65b93012de
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ms-file-size-benchmark-comparison

## Summary

Benchmarks the storage footprint of MZA's HDF5-based MS data format against competing open formats (mz5, mzXML, mzML, mzMLb) by converting identical vendor raw files under lossless, zero-intensity-omitted settings and recording the resulting byte sizes.

## When to use

Use when choosing an open mass spectrometry file format for storing converted vendor raw data and you need an empirical file-size comparison across formats like MZA, mz5, mzXML, mzML and mzMLb.

## When NOT to use

- You need speed/parse-time benchmarks rather than storage footprint - the article reports improvements in speed are still needed for MZA and does not provide runtime benchmarks for this comparison.
- Your comparison requires identical conversion parameters across formats that you cannot reproduce (e.g. no ProteoWizard version control), since the size ordering depends on equivalent lossless settings.
- You are converting only a single small file; the benchmark's discriminative signal comes from datasets with large numbers of spectra such as the LC-SLIM-MS file.

## Inputs

- Three vendor raw MS files from MassIVE MSV000090269: LC-DIA proteomics .raw (60 min), LC-DTIM lipidomics raw (30 min), LC-SLIM proteomics raw (60 min)
- MZA executable (Windows or Wine/Docker-wrapped)
- ProteoWizard toolkit (mz5, mzXML, mzML, mzMLb converters)

## Outputs

- MZA files (.mza) for each raw dataset
- mz5, mzXML, mzML and mzMLb conversions of each raw dataset
- file_size_comparison.csv with per-dataset, per-format byte sizes (3 datasets x 5 formats)

## How to apply

Download the three representative vendor raw files from the MassIVE MSV000090269 deposit: a 60 min LC-DIA proteomics .raw file, a 30 min LC-DTIM lipidomics file, and a 60 min LC-SLIM proteomics file. Convert each raw file to MZA by running the MZA executable (mza -file <raw> with no -intensityThreshold argument, so conversion is lossless with zero-intensity peaks omitted) on Windows, under Wine 6.0.3 + wine-mono 7.3.0 on Ubuntu 22.04, or inside the mzacontainer Docker image with WINEDEBUG=-all. In parallel, convert the same raw files to mz5, mzXML, mzML and mzMLb with the ProteoWizard toolkit, likewise omitting zero-intensity peaks, so all five formats are compared on equivalent lossless content. Record the byte size of every output file and assemble a per-dataset, per-format size table (file_size_comparison.csv). The rationale is that MZA's jagged HDF5 arrays omit zero-intensity values, so the comparison isolates format-level compression efficiency rather than data-selection choices; interpret results relative to the largest-spectra-count file, where format differences are most pronounced.

## Related tools

- **MZA** (Converts vendor raw MS files (.d, .raw) to the HDF5-based MZA file structure, omitting zero-intensity peaks, to be size-benchmarked against other formats.) — https://github.com/PNNL-m-q/mza
- **ProteoWizard** (Generates the comparison formats mz5, mzXML, mzML and mzMLb from the same vendor raw files under equivalent lossless conversion settings.)
- **HDF5** (Underlying binary hierarchical format that stores MZA's metadata table and jagged arrays_intensity/arrays_mz (+arrays_mzbin) datasets.)

## Examples

```
docker run -it --rm -e WINEDEBUG=-all -v test_data:/data mzacontainer wine mza.exe -file /data/LCMSMS_Lipids_POS.raw
```

## Evaluation signals

- file_size_comparison.csv contains a nonzero size entry for every dataset-format combination (3 datasets x 5 formats).
- Relative ordering reproduces the paper's result: MZA smaller than mzXML and mzML, larger than mz5 and mzMLb, with mz5 smallest for all files.
- The LC-SLIM proteomics file (largest number of spectra) shows the most pronounced size differences between formats.
- All conversions used lossless settings with zero-intensity peaks removed (no -intensityThreshold supplied to MZA), so sizes reflect format compression, not data filtering.

## Limitations

- MZA files are larger than mz5 and mzMLb - mz5 provided the smallest output for all files - so MZA is not the optimal choice when storage is the sole criterion.
- Improvements in speed and file size are still needed for MZA, and the paper provides no quantitative runtime benchmarks for conversion speed.
- The MZA executable currently runs only on Windows natively; on Unix-like systems it requires Wine 6.0.3 + wine-mono 7.3.0 or the mzacontainer Docker image.
- No version number is stated for the MZA tool, so exact reproducibility of size results across MZA releases is not guaranteed.

## Evidence

- [results] MZA files were larger than mz5 and mzMLb, and smaller than mzXML and mzML, most notably for the LC-SLIM-MS file with the largest number of spectra: [evidence span withheld — anchor status: not_found]
