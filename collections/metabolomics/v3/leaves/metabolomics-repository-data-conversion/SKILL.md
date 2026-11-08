---
name: metabolomics-repository-data-conversion
description: Use when a study from MetaboLights or Metabolomics Workbench (e.g. an
  NMR study like MTBLS563 with Bruker fid/acqu files) must be transformed into an
  analysis-ready format before statistical analysis or ROC/AUC evaluation.
when_to_use_negative:
- The input is already a processed feature table or the csv output of a prior CloMet
  conversion — re-running conversion adds no value and conversion is not a data-processing
  step
- The goal is to apply data-processing algorithms themselves (e.g. peak picking or
  normalization); CloMet explicitly leaves processing out of scope for other developers
- Class metadata is not available alongside the metabolomics data in the repository,
  making classification-oriented re-analysis infeasible regardless of conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Detects the repository file-system format and converts raw/NMR data into the
    custom directory-based standard and analysis-ready csv output
  repo: github.com/rmallol/clomet
- name: Docker
  role: Runtime container through which CloMet is installed and executed
- name: MetaboAnalyst
  role: Downstream web-based platform into which the converted csv is uploaded for
    Statistical Analysis (e.g. ROC/AUC class discrimination)
- name: Workflows4Metabolomics
  role: Alternative downstream platform consuming the converted format via NMR_Read,
    NMR_Preprocessing, or NMR_Bucketing
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:686db563f702f76feee354621eba3539d79be2c456735cac70463280e3c391d5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-repository-data-conversion

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Convert raw and NMR-based metabolomics data sets from MetaboLights or Metabolomics Workbench into CloMet's standardized directory-based format, ready for direct loading into MetaboAnalyst or Workflows4Metabolomics. This fills the data-accessibility gap between public repositories and analysis platforms.

## When to use

Use when a study from MetaboLights or Metabolomics Workbench (e.g. an NMR study like MTBLS563 with Bruker fid/acqu files) must be transformed into an analysis-ready format before statistical analysis or ROC/AUC evaluation.

## When NOT to use

- The input is already a processed feature table or the csv output of a prior CloMet conversion — re-running conversion adds no value and conversion is not a data-processing step
- The goal is to apply data-processing algorithms themselves (e.g. peak picking or normalization); CloMet explicitly leaves processing out of scope for other developers
- Class metadata is not available alongside the metabolomics data in the repository, making classification-oriented re-analysis infeasible regardless of conversion

## Inputs

- Study identifier for a MetaboLights or Metabolomics Workbench data set (e.g. MTBLS563)
- Raw NMR or metabolomics files in any of 11 supported formats: .fid, .mzXML, .CNX, .nmrML, .TABULAR single files, or Bruker directory layouts
- Users' own private/local data via CloMet's LOCAL tab

## Outputs

- Ready-to-use csv in CloMet's standardized output format (e.g. for MetaboAnalyst upload)
- Custom directory-based standard: study directory with a_, i_, m_, s_, audit metadata files and per-sample subdirectories with acqu, acqus, fid files and a pdata directory
- Invalid/unconvertible study flags for studies lacking a 1r file or with an undetectable format

## How to apply

Retrieve the study files from the repository using the study identifier (e.g. MTBLS563), then run CloMet — available through a Docker file — on the study id so it emits a ready-to-use csv in CloMet's standardized output format with minimal interaction. CloMet's DetectFormat module classifies the input by file-system structure (not file names) against 11 supported format signatures: single files with .fid, .mzXML, .CNX, .nmrML, or .TABULAR extensions, plus Bruker directories distinguished by their internal folder layout; studies lacking a 1r file in any sample are flagged invalid. The custom standard output is a directory per study with study ID and metadata files (a_, i_, m_, s_, audit) at the top level and per-sample subdirectories containing experiment directories with acqu, acqus, fid files and a pdata directory. Inspect the converted output to confirm sample rows and class labels (e.g. bacterial, viral, control) were preserved, then upload it into MetaboAnalyst's Statistical Analysis module or run Workflows4Metabolomics tools such as NMR_Read, NMR_Preprocessing, or NMR_Bucketing.

## Related tools

- **CloMet** (Detects the repository file-system format and converts raw/NMR data into the custom directory-based standard and analysis-ready csv output) — github.com/rmallol/clomet
- **Docker** (Runtime container through which CloMet is installed and executed)
- **MetaboAnalyst** (Downstream web-based platform into which the converted csv is uploaded for Statistical Analysis (e.g. ROC/AUC class discrimination))
- **Workflows4Metabolomics** (Alternative downstream platform consuming the converted format via NMR_Read, NMR_Preprocessing, or NMR_Bucketing)

## Evaluation signals

- Every converted study contains a 1r file in each sample's experiment directory; any study missing the 1r file is flagged invalid
- Converted study directory matches the custom standard: study ID plus a_, i_, m_, s_, audit metadata files at top level, per-sample subdirectories with acqu, acqus, fid files and a pdata directory
- Sample rows and class labels (e.g. bacterial, viral, control) are preserved in the converted csv before upload to MetaboAnalyst
- Downstream reproduction check: for MTBLS563, ROC/AUC values after conversion match published results of 0.82 (bacterial vs control) and 0.84 (viral vs control)

## Limitations

- Studies with an undetectable format or missing the 1r file in any sample cannot be converted and are left untransformed
- CloMet does not implement data-processing algorithms; processing is out of scope and left to other developers
- Re-analysis for classification problems is infeasible when class information is not available alongside the data in the public repository
- Maintenance and extension depend on the open-source developer community; integration with existing analysis platforms remains a future plan, not implemented

## Evidence
