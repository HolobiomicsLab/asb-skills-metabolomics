---
name: nmr-study-file-validation
description: Use when a study downloaded from MetaboLights or Metabolomics Workbench
  has been converted to CloMet's internal standard (a_ / i_ / m_ / s_ / audit metadata
  files with per-sample experiment directories holding acqu, acqus, fid) and you must
  decide whether it is usable downstream.
when_to_use_negative:
- The input is already a processed/analysis-ready file (e.g. a bucketed or preprocessed
  NMR matrix for MetaboAnalyst) — validation of raw repository file trees does not
  apply
- You need spectral data processing algorithms (preprocessing, bucketing); CloMet
  explicitly leaves these out of scope
- The data is the user's own private/local data being translated via the LOCAL tab
  where class metadata or format differs from the supported repository signatures
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: CloMet
  role: Detects the input format (11 signatures) and validates that each converted
    study is a valid custom-standard directory with a 1r file in every sample
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Runtime environment in which CloMet (and its validation stage) is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:e39ee4921b1482d779338908daafa3c804828da982e6692c74d451f77ef75486
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmr-study-file-validation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Validates NMR metabolomics studies converted to CloMet's custom directory-based standard format by checking structural invariants — chiefly that every per-sample subdirectory contains a 1r file — and flagging studies that fail detection/conversion as invalid.

## When to use

Use when a study downloaded from MetaboLights or Metabolomics Workbench has been converted to CloMet's internal standard (a_ / i_ / m_ / s_ / audit metadata files with per-sample experiment directories holding acqu, acqus, fid) and you must decide whether it is usable downstream.

## When NOT to use

- The input is already a processed/analysis-ready file (e.g. a bucketed or preprocessed NMR matrix for MetaboAnalyst) — validation of raw repository file trees does not apply
- You need spectral data processing algorithms (preprocessing, bucketing); CloMet explicitly leaves these out of scope
- The data is the user's own private/local data being translated via the LOCAL tab where class metadata or format differs from the supported repository signatures

## Inputs

- Converted or unconverted NMR study directory tree from MetaboLights or Metabolomics Workbench (Bruker folders, .fid, .mzXML, .CNX, .nmrML, or .TABULAR files)
- CloMet custom standard format study directory (study ID with a_, i_, m_, s_, audit metadata files and per-sample subdirectories containing acqu, acqus, fid files and a pdata directory)

## Outputs

- Valid converted study directory tree in the custom standard format
- Flags or list of invalid studies (missing 1r file, undetectable format, or failed conversion), left untransformed

## How to apply

After CloMet's DetectFormat module classifies the study against the 11 supported format signatures (single files with .fid, .mzXML, .CNX, .nmrML, or .TABULAR extensions, or Bruker directories identified by internal folder layout), walk the converted directory tree and verify the custom-standard layout: study ID directory containing a_, i_, m_, s_ and audit metadata files at the top level, per-sample subdirectories with experiment directories holding acqu, acqus, and fid files plus a pdata directory. Structure matching must rely on the file system layout rather than file names, since CloMet reads only structure. Apply the decisive invariant: a study lacking a 1r file in any sample is considered invalid and should be flagged as such; studies that cannot be detected or converted are left untransformed rather than partially emitted. Emit the valid converted directory tree as output and record invalid studies so they are not passed to analysis platforms like MetaboAnalyst or Workflows4Metabolomics.

## Related tools

- **CloMet** (Detects the input format (11 signatures) and validates that each converted study is a valid custom-standard directory with a 1r file in every sample) — https://github.com/rmallol/clomet
- **Docker** (Runtime environment in which CloMet (and its validation stage) is executed)

## Evaluation signals

- Every valid output study directory contains a 1r file in each per-sample experiment directory
- Top level of each output study directory contains the a_, i_, m_, s_, and audit metadata files
- Each per-sample subdirectory contains experiment directories with acqu, acqus, fid files and a pdata directory
- Studies classified as undetectable or unconvertible are left untransformed and reported as invalid rather than silently emitted
- Format classification matches one of the 11 supported signatures (5 single-file extensions plus Bruker directory layouts) rather than being inferred from file names

## Limitations

- A study missing the 1r file in any single sample is rejected as invalid, so one bad sample invalidates the whole study
- If class information is not available alongside the metabolomics data in public repositories, re-analysis for classification problems is not feasible even after validation
- Maintenance and extension of the validation logic depends on the open-source developer community, and integration with existing analysis platforms remains a future plan

## Evidence

- [full_text] any study lacking the 1r file in any sample is considered invalid: [evidence span withheld — anchor status: not_found]
