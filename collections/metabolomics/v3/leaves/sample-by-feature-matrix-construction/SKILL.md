---
name: sample-by-feature-matrix-construction
description: Use when raw NMR metabolomics data (fid, 1r, acqu/acqus files) from a
  repository study must be converted into a binned sample-by-variable matrix with
  a fixed spectral resolution (1000 columns) for tools like MetaboAnalyst.
when_to_use_negative:
- The data are already a feature/binned table ready for MetaboAnalyst or Workflows4Metabolomics
  — re-binning would lose or distort information
- The data are MS-based (GC–MS/LC–MS) rather than NMR-based, since CloMet focuses
  on NMR-based metabolomics
- Class metadata is unavailable and the goal is classification re-analysis, which
  is infeasible regardless of matrix construction
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0593
tools:
- name: CloMet
  role: Platform whose ImportData module reads standardized per-sample NMR files,
    builds the sample-by-spectral-point matrix, and applies binning to 1000 columns
  repo: github.com/rmallol/clomet
- name: MetaboAnalyst
  role: Downstream web-based analysis platform that consumes the binned data matrix
    output
- name: Workflows4Metabolomics
  role: Downstream workflow platform whose NMR_Read/NMR_Preprocessing/NMR_Bucketing
    tools require the ready-to-use format produced here
- name: Docker
  role: Execution environment for running CloMet to produce the binned matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:ba61b74f1f12827ea75484d9411232ea85e240c98ffe6e009346fc71d8aba41e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# sample-by-feature-matrix-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstructs CloMet's ImportData module: reads per-sample NMR acquisition files (acqu/acqus, fid, 1r) from CloMet's internal directory-based standard and assembles a sample-by-spectral-point data matrix, then applies uniform binning to reduce the spectral axis to exactly 1000 columns for downstream analysis platforms.

## When to use

Use when raw NMR metabolomics data (fid, 1r, acqu/acqus files) from a repository study must be converted into a binned sample-by-variable matrix with a fixed spectral resolution (1000 columns) for tools like MetaboAnalyst.

## When NOT to use

- The data are already a feature/binned table ready for MetaboAnalyst or Workflows4Metabolomics — re-binning would lose or distort information
- The data are MS-based (GC–MS/LC–MS) rather than NMR-based, since CloMet focuses on NMR-based metabolomics
- Class metadata is unavailable and the goal is classification re-analysis, which is infeasible regardless of matrix construction

## Inputs

- Standardized per-sample NMR acquisition files (acqu/acqus, fid, 1r) in CloMet's internal directory-based format
- Study directory previously harmonized from MetaboLights or Metabolomics Workbench
- Spectral metadata parsed from acqu/acqus parameter files

## Outputs

- binned_data_matrix.csv — sample-by-1000-binned-column numeric matrix
- Intermediate sample-by-spectral-point data matrix (one row per sample, one column per spectral variable)

## How to apply

First scan the standardized study directory (CloMet's custom internal standard that harmonizes MetaboLights/Metabolomics Workbench layouts) and enumerate per-sample NMR files: acqu/acqus parameter files, raw fid, and processed 1r spectra. Parse acqu/acqus to obtain spectral metadata needed for signal reconstruction, then read the 1r spectral data for every sample and assemble rows (samples) by columns (spectral points) into a numeric data matrix. Apply uniform binning (bucketing) over the spectral-point axis so the final matrix contains exactly 1000 data columns, as the paper specifies. Write the result to binned_data_matrix.csv in a format consumable by downstream platforms such as MetaboAnalyst or Workflows4Metabolomics (e.g. NMR_Read, NMR_Preprocessing, NMR_Bucketing).

## Related tools

- **CloMet** (Platform whose ImportData module reads standardized per-sample NMR files, builds the sample-by-spectral-point matrix, and applies binning to 1000 columns) — github.com/rmallol/clomet
- **MetaboAnalyst** (Downstream web-based analysis platform that consumes the binned data matrix output)
- **Workflows4Metabolomics** (Downstream workflow platform whose NMR_Read/NMR_Preprocessing/NMR_Bucketing tools require the ready-to-use format produced here)
- **Docker** (Execution environment for running CloMet to produce the binned matrix)

## Evaluation signals

- Output file binned_data_matrix.csv exists and is loadable as a numeric table
- The spectral dimension of the output matrix equals exactly 1000 binned columns, matching the paper's specified binning resolution
- Number of rows equals the number of samples with 1r files in the standardized study directory
- Downstream tools (MetaboAnalyst, Workflows4Metabolomics NMR_Bucketing) accept the file format without conversion
- Each row's spectral content traces back to that sample's 1r data with no missing samples

## Limitations

- Binning resolution is fixed at exactly 1000 columns per the paper; other bucketing widths are not produced by this path
- If class metadata is not available alongside the metabolomics data in public repositories, re-analysis oriented to classification problems will not be feasible even with a valid matrix
- CloMet itself does not implement further data processing algorithms; processing beyond matrix construction is out of its scope and left to other developers

## Evidence

- [full_text] a data binning technique is then implemented so the final binned data set contains exactly 1000 data columns, based on the 1r file spectral data: [evidence span withheld — anchor status: not_found]
- [full_text] the ImportData module reads the necessary files from every sample to build a data matrix where each row is a sample and each column is a spectral variable: [evidence span withheld — anchor status: not_found]
