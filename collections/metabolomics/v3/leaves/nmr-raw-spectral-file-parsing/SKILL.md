---
name: nmr-raw-spectral-file-parsing
description: Use when raw NMR-based metabolomics files (acqu/acqus, fid, 1r) from
  MetaboLights or Metabolomics Workbench studies must be reconstructed into a sample-by-spectral-point
  matrix and binned to exactly 1000 columns.
when_to_use_negative:
- The input is already a binned/feature table ready for statistical analysis — parsing
  and binning would be redundant
- The study is not NMR-based (e.g. GC–MS or LC–MS raw data), since CloMet focuses
  on NMR-based metabolomics
- You need actual data-processing algorithms (e.g. advanced preprocessing) — CloMet
  leaves these out of scope for other developers
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Runs the ImportData module that reads acqu/acqus, fid, and 1r files, builds
    the sample-by-spectral-point matrix, and applies binning to 1000 columns
  repo: github.com/rmallol/clomet
- name: Docker
  role: CloMet is distributed as a Docker file, used to run the parsing/binning pipeline
    on the standardized study
- name: NMR_Read
  role: Downstream Workflows4Metabolomics tool that can consume the format produced
    by this parsing step
- name: NMR_Bucketing
  role: Downstream Workflows4Metabolomics bucketing tool compatible with the binned
    output format
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:e7691e01e1c3afa59daac2b3110326b8dfd294c7b8b8e6b56aa74e17be85d8e4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - NMR_Bucketing
  - NMR_Read
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# nmr-raw-spectral-file-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Parsing raw NMR metabolomics acquisition files (acqu/acqus, fid, 1r) into a sample-by-spectral-point numeric data matrix, then applying binning so the final matrix has exactly 1000 spectral data columns, following CloMet's ImportData module.

## When to use

Use when raw NMR-based metabolomics files (acqu/acqus, fid, 1r) from MetaboLights or Metabolomics Workbench studies must be reconstructed into a sample-by-spectral-point matrix and binned to exactly 1000 columns.

## When NOT to use

- The input is already a binned/feature table ready for statistical analysis — parsing and binning would be redundant
- The study is not NMR-based (e.g. GC–MS or LC–MS raw data), since CloMet focuses on NMR-based metabolomics
- You need actual data-processing algorithms (e.g. advanced preprocessing) — CloMet leaves these out of scope for other developers

## Inputs

- Standardized per-sample NMR acquisition files (acqu/acqus) from a CloMet-harmonized study directory
- Raw FID signal files (fid) per sample
- Processed NMR spectra (1r files) per sample
- Study identifier from MetaboLights or Metabolomics Workbench resolving to the standardized study directory

## Outputs

- Sample-by-spectral-point numeric data matrix (one row per sample, one column per spectral variable)
- Binned data matrix with exactly 1000 spectral columns (e.g. binned_data_matrix.csv) consumable by MetaboAnalyst or Workflows4Metabolomics

## How to apply

Scan the standardized study directory — CloMet's internal directory-based standard that harmonizes the different formats found in public repositories — and enumerate each sample's acquisition files (acqu/acqus), raw fid, and processed 1r files. Parse the acqu/acqus parameter files to obtain the spectral metadata needed for signal reconstruction, then read every sample's fid and 1r signals and assemble them as rows of a sample-by-spectral-point numeric data matrix (one row per sample, one column per spectral variable). Apply uniform binning (bucketing) over the spectral-point axis so the binned data set contains exactly 1000 data columns based on the 1r spectral data, as the paper specifies. Write the binned sample-by-1000-column matrix (e.g. binned_data_matrix.csv) in a format directly consumable by MetaboAnalyst or Workflows4Metabolomics tools such as NMR_Read, NMR_Preprocessing, or NMR_Bucketing. Rationale: raw repository data lack standardized file-system formats, so reconstruction must happen after CloMet's harmonization step for parsing to succeed across all samples.

## Related tools

- **CloMet** (Runs the ImportData module that reads acqu/acqus, fid, and 1r files, builds the sample-by-spectral-point matrix, and applies binning to 1000 columns) — github.com/rmallol/clomet
- **Docker** (CloMet is distributed as a Docker file, used to run the parsing/binning pipeline on the standardized study)
- **NMR_Read** (Downstream Workflows4Metabolomics tool that can consume the format produced by this parsing step)
- **NMR_Bucketing** (Downstream Workflows4Metabolomics bucketing tool compatible with the binned output format)

## Evaluation signals

- The output matrix exists and its spectral dimension equals exactly 1000 binned columns, matching the paper's specified binning resolution
- The data matrix has one row per sample in the standardized study directory, with no samples dropped during file enumeration
- Parsing succeeded on every sample's acqu/acqus, fid, and 1r files (no missing or unparseable acquisition files)
- The binned matrix loads directly in MetaboAnalyst or Workflows4Metabolomics (NMR_Read/NMR_Bucketing) without format conversion
- The binned values derive from the 1r processed spectra, not the raw fid alone

## Limitations

- Binning is fixed at exactly 1000 data columns per the paper's specification; other resolutions are not covered by this method
- If class metadata is unavailable alongside the metabolomics data in public repositories, downstream re-analysis for classification problems will not be feasible even after successful parsing and binning
- CloMet's parsing depends on data first being harmonized into its internal directory-based standard; raw, unhomogenized repository layouts will not parse
- CloMet itself does not implement data processing algorithms beyond this conversion — this is out of its scope, and maintenance relies on the open-source developer community

## Evidence

- [results] a data binning technique is then implemented so the final binned data set contains exactly 1000 data columns, based on the 1r file spectral data.: [evidence span withheld — anchor status: not_found]
