---
name: bruker-directory-layout-parsing
description: Use when a downloaded study from MetaboLights or Metabolomics Workbench
  must be format-classified by scanning extensions and directory structure (e.g. Bruker
  folders with acqu/acqus/fid) before conversion to a standard layout.
when_to_use_negative:
- The study is already in CloMet's custom directory-based standard format — re-detection
  and re-conversion is unnecessary
- The goal is downstream data processing or statistical analysis — CloMet itself does
  not implement data processing algorithms; that is out of its scope and left to tools
  like MetaboAnalyst or Workflows4Metabolomics
- The input uses a format outside the 11 supported signatures (single .fid/.mzXML/.CNX/.nmrML/.TABULAR
  files or recognized Bruker directory layouts) — such studies will be flagged invalid,
  not transformed
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: CloMet
  role: Implements the DetectFormat module that classifies the study format and dispatches
    it to the converter emitting the custom directory-based standard format
  repo: github.com/rmallol/clomet
- name: Docker
  role: Runtime packaging through which CloMet is made available for running the detection
    and conversion stage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:8b5888603c681b04cbc98f456b1628e80e1f7dceb7fb21c258c6416198df9d27
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bruker-directory-layout-parsing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Detects the file-system format of raw NMR metabolomics studies — including Bruker directories distinguished by internal folder layout rather than file names — and transforms them into CloMet's custom directory-based standard format.

## When to use

Use when a downloaded study from MetaboLights or Metabolomics Workbench must be format-classified by scanning extensions and directory structure (e.g. Bruker folders with acqu/acqus/fid) before conversion to a standard layout.

## When NOT to use

- The study is already in CloMet's custom directory-based standard format — re-detection and re-conversion is unnecessary
- The goal is downstream data processing or statistical analysis — CloMet itself does not implement data processing algorithms; that is out of its scope and left to tools like MetaboAnalyst or Workflows4Metabolomics
- The input uses a format outside the 11 supported signatures (single .fid/.mzXML/.CNX/.nmrML/.TABULAR files or recognized Bruker directory layouts) — such studies will be flagged invalid, not transformed

## Inputs

- Downloaded study file trees from MetaboLights or Metabolomics Workbench
- Single-file formats: .fid, .mzXML, .CNX, .nmrML, .TABULAR files
- Bruker directory formats with internal acqu, acqus, fid files and a pdata directory

## Outputs

- Converted study directory tree in CloMet's custom standard format (study ID, a_/i_/m_/s_/audit metadata files, per-sample subdirectories with experiment dirs containing acqu, acqus, fid and pdata)
- Invalid-study flags for studies lacking a 1r file or with undetectable/unconvertible formats

## How to apply

Scan the downloaded study's file tree and inspect file extensions and directory structure to classify the format, reading only the file structure rather than file names. Match the observed layout against the 11 supported format signatures: single files with .fid, .mzXML, .CNX, .nmrML, or .TABULAR extensions, and Bruker directories distinguished by their internal folder layout. Dispatch the detected format to the corresponding converter routine that emits the custom standard format in which each study is a directory with a study ID, metadata files (a_, i_, m_, s_, audit) at the top level, and per-sample subdirectories containing experiment directories with acqu, acqus, fid files and a pdata directory. Validate each converted study by checking for the presence of a 1r file; flag studies that cannot be detected or converted as invalid.

## Related tools

- **CloMet** (Implements the DetectFormat module that classifies the study format and dispatches it to the converter emitting the custom directory-based standard format) — github.com/rmallol/clomet
- **Docker** (Runtime packaging through which CloMet is made available for running the detection and conversion stage)

## Evaluation signals

- Every converted sample subdirectory contains a 1r file — any study lacking the 1r file in any sample is considered invalid
- Converted study directories contain top-level metadata files named with the a_, i_, m_, s_, and audit prefixes
- Each sample subdirectory contains experiment directories holding acqu, acqus, fid files and a pdata directory
- Studies whose layout matches no supported signature are flagged invalid rather than silently transformed
- Detection is based on directory structure and extensions only — renaming files without changing structure should not change the detected format

## Limitations

- Only 11 format signatures are supported (single .fid, .mzXML, .CNX, .nmrML, .TABULAR files and Bruker directory layouts); undetectable formats are left untransformed
- Any study lacking the 1r file in any sample is considered invalid and cannot be used downstream
- CloMet depends on the open-source developer community to extend and maintain it, and support for new formats relies on community contributions

## Evidence

- [other] any study lacking the 1r file in any sample is considered invalid: [evidence span withheld — anchor status: not_found]
- [other] CloMet reads only the file structure rather than file names: [evidence span withheld — anchor status: not_found]
