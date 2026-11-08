---
name: nmr-spectra-format-conversion
description: Use when raw NMR metabolomics data pulled from MetaboLights or Metabolomics
  Workbench must be reshaped from repository-specific layouts (.fid, .mzXML, .CNX,
  .nmrML, .TABULAR, Bruker directories) into an analysis-ready standard format.
when_to_use_negative:
- The user's own private/local data being translated via the LOCAL tab follows a bespoke
  layout rather than a public-repository export signature
- The goal is downstream data processing (preprocessing, bucketing, statistical analysis)
  — CloMet deliberately implements no processing algorithms, that is out of its scope
- The input is already in the custom standard format with the 1r file present in every
  sample — no conversion is needed
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: CloMet
  role: Performs format detection against the 11 supported format signatures and dispatches
    each detected format to its converter routine that emits the custom directory-based
    standard; Docker-based and open source
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Container runtime in which the CloMet detection/conversion pipeline is executed
  repo: https://github.com/rmallol/clomet
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:6bcb6ff2c5c87afb64e88db8a93542c85387acb61bf7b11186b97e16b1ca46a8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboLights
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# nmr-spectra-format-conversion

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Detects the heterogeneous file-system formats of NMR metabolomics studies downloaded from MetaboLights and Metabolomics Workbench and converts them into CloMet's custom directory-based standard format that analysis platforms such as MetaboAnalyst and Workflows4Metabolomics can consume directly.

## When to use

Use when raw NMR metabolomics data pulled from MetaboLights or Metabolomics Workbench must be reshaped from repository-specific layouts (.fid, .mzXML, .CNX, .nmrML, .TABULAR, Bruker directories) into an analysis-ready standard format.

## When NOT to use

- The user's own private/local data being translated via the LOCAL tab follows a bespoke layout rather than a public-repository export signature
- The goal is downstream data processing (preprocessing, bucketing, statistical analysis) — CloMet deliberately implements no processing algorithms, that is out of its scope
- The input is already in the custom standard format with the 1r file present in every sample — no conversion is needed

## Inputs

- NMR metabolomics study directories or raw files downloaded from MetaboLights or Metabolomics Workbench (single .fid/.mzXML/.CNX/.nmrML/.TABULAR files or Bruker directory trees)

## Outputs

- Converted study directory tree in the custom standard format (study ID directory with a_, i_, m_, s_, audit metadata files at top level and per-sample subdirectories containing acqu, acqus, fid files and a pdata directory)
- Invalid/unconvertible studies left untransformed and flagged

## How to apply

Run CloMet (via the provided Docker container) on a downloaded study: first scan the study's file tree and classify its format against the 11 supported signatures — single files ending in .fid, .mzXML, .CNX, .nmrML, or .TABULAR, plus Bruker directories recognized by their internal folder layout rather than file names. Dispatch the detected format to the matching converter routine, which emits a custom standard in which each study is a directory named by study ID containing top-level metadata files (a_, i_, m_, s_, audit) and per-sample subdirectories holding experiment directories with acqu, acqus, fid files and a pdata directory. Validate each converted study by checking that every sample contains a 1r file; studies that cannot be detected or converted, or that lack the 1r file in any sample, are flagged invalid and left untransformed.

## Related tools

- **CloMet** (Performs format detection against the 11 supported format signatures and dispatches each detected format to its converter routine that emits the custom directory-based standard; Docker-based and open source) — https://github.com/rmallol/clomet
- **Docker** (Container runtime in which the CloMet detection/conversion pipeline is executed) — https://github.com/rmallol/clomet

## Evaluation signals

- Each converted study directory contains the a_, i_, m_, s_ and audit metadata files at its top level, named by study ID
- Every sample subdirectory in the converted tree contains an experiment directory with acqu, acqus, fid files and a pdata directory
- A 1r file is present in every sample; absence in any sample invalidates the study
- Benchmark expectation: whole-process correct-output rates of 38/41 (92.7%) for accepted data sets and 38/52 (73.1%) for all 52 repository studies
- Detection relies only on file structure, not file names — renaming files must not change the detected format

## Limitations

- Studies lacking the 1r file in any sample are considered invalid and are left untransformed
- Studies whose format cannot be detected or converted are flagged invalid rather than forced through conversion
- Only the 11 supported format signatures (5 single-file extensions plus Bruker directory layouts) can be converted; other repository layouts fail
- CloMet does not itself implement data processing algorithms — conversion only harmonizes file systems for downstream platforms

## Evidence
