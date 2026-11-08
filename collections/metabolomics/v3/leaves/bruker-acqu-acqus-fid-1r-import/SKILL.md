---
name: bruker-acqu-acqus-fid-1r-import
description: Use when a standardized CloMet study directory contains per-sample NMR
  files (acqu/acqus, fid, 1r) and a sample-by-variable matrix binned to 1000 spectral
  columns is needed for downstream analysis platforms.
when_to_use_negative:
- The data are already in a format directly usable by MetaboAnalyst or Workflows4Metabolomics
  (e.g. NMR_Read/NMR_Bucketing compatible) — conversion/import is unnecessary
- The task requires spectral data-processing algorithms beyond import and binning
  — CloMet explicitly leaves processing algorithms out of scope for other developers
- The input is MS-based (GC–MS/LC–MS) raw data outside CloMet's NMR-focused harmonization,
  or no class metadata accompanies the study and classification-oriented re-analysis
  is the goal
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CloMet
  role: Implements the ImportData module that reads acqu/acqus, fid and 1r files from
    every sample, builds the sample-by-spectral-point matrix, and applies binning
    to 1000 columns
  repo: github.com/rmallol/clomet
- name: NMR_Read
  role: Downstream Workflows4Metabolomics tool that consumes the format produced by
    this skill
- name: NMR_Bucketing
  role: Workflows4Metabolomics bucketing tool whose required input format the binned_data_matrix.csv
    output targets
- name: Docker
  role: Packaging/delivery mechanism for running CloMet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:0890e8ed2eaf377964493f411da42cf51412350b094a7fed443e30996bfeed19
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - NMR_Bucketing
  - NMR_Read
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bruker-acqu-acqus-fid-1r-import

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

CloMet's ImportData skill reads per-sample Bruker NMR acquisition files (acqu/acqus, fid, 1r) that have been harmonized into CloMet's internal directory-based standard and assembles a sample-by-spectral-point data matrix. A binning (bucketing) step is then applied over the spectral axis so the final binned data set contains exactly 1000 data columns derived from the 1r spectral data.

## When to use

Use when a standardized CloMet study directory contains per-sample NMR files (acqu/acqus, fid, 1r) and a sample-by-variable matrix binned to 1000 spectral columns is needed for downstream analysis platforms.

## When NOT to use

- The data are already in a format directly usable by MetaboAnalyst or Workflows4Metabolomics (e.g. NMR_Read/NMR_Bucketing compatible) — conversion/import is unnecessary
- The task requires spectral data-processing algorithms beyond import and binning — CloMet explicitly leaves processing algorithms out of scope for other developers
- The input is MS-based (GC–MS/LC–MS) raw data outside CloMet's NMR-focused harmonization, or no class metadata accompanies the study and classification-oriented re-analysis is the goal

## Inputs

- Standardized CloMet study directory (internal directory-based standard) containing per-sample NMR files: acqu/acqus parameter files, fid raw signal files, and 1r processed spectra
- Study identifier for a MetaboLights or Metabolomics Workbench data set already harmonized by CloMet's standardization step

## Outputs

- Sample-by-spectral-point numeric data matrix (rows = samples, columns = spectral variables)
- binned_data_matrix.csv with exactly 1000 binned spectral columns per sample, consumable by MetaboAnalyst or Workflows4Metabolomics

## How to apply

First confirm the study directory has been standardized into CloMet's internal directory-based format, since raw repository layouts from MetaboLights and Metabolomics Workbench are not directly readable. Enumerate every sample's acquisition files, parse the acqu/acqus parameter files for spectral metadata needed for signal reconstruction, then read each sample's raw fid and processed 1r signal files. Assemble these into a numeric matrix where each row is a sample and each column is a spectral variable. Finally apply uniform binning (bucketing) over the spectral-point axis — based on the 1r file spectral data — reducing each spectrum to exactly 1000 data columns, matching the paper's specified binning resolution. Write the result as binned_data_matrix.csv in a format consumable by downstream tools such as Workflows4Metabolomics's NMR_Read/NMR_Bucketing or MetaboAnalyst.

## Related tools

- **CloMet** (Implements the ImportData module that reads acqu/acqus, fid and 1r files from every sample, builds the sample-by-spectral-point matrix, and applies binning to 1000 columns) — github.com/rmallol/clomet
- **NMR_Read** (Downstream Workflows4Metabolomics tool that consumes the format produced by this skill)
- **NMR_Bucketing** (Workflows4Metabolomics bucketing tool whose required input format the binned_data_matrix.csv output targets)
- **Docker** (Packaging/delivery mechanism for running CloMet)

## Evaluation signals

- The output binned_data_matrix.csv exists and its spectral dimension equals exactly 1000 binned columns, matching the paper's specified binning resolution
- The number of matrix rows equals the number of samples enumerated in the standardized study directory (no samples dropped or duplicated)
- Each sample's spectral values trace back to that sample's 1r processed signal file, with binning applied over the spectral-point axis
- The output is loadable by downstream platforms as a MetaboAnalyst- or Workflows4Metabolomics-compatible data set (e.g. via NMR_Read/NMR_Bucketing)
- All per-sample acquisition parameter files (acqu/acqus) parsed without missing-sample errors across the standardized directory

## Limitations

- CloMet does not implement data processing algorithms itself; this skill covers import/assembly/binning only, and processing is left to other developers
- If class information is not available alongside the metabolomics data in public repositories, downstream re-analysis oriented to classification problems will not be feasible
- Raw data from repositories must first be standardized into CloMet's internal directory-based format; non-standardized file system layouts cannot be read directly
- Maintenance and extension rely on the open-source developer community; no software version number is stated in the discussion

## Evidence
