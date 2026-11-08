---
name: file-system-format-detection
description: Use when a downloaded metabolomics study from MetaboLights or Metabolomics
  Workbench arrives in an unknown raw layout and it must be classified into a known
  format signature before transformation into an analysis-ready standard format.
when_to_use_negative:
- The data are already in the custom CloMet standard format or directly consumable
  by MetaboAnalyst or Workflows4Metabolomics — no format detection is needed.
- CloMet itself does not implement data processing algorithms, so this skill cannot
  be used for downstream spectral processing such as the NMR_Read, NMR_Preprocessing,
  or NMR_Bucketing steps.
- Class metadata is absent from the repository data set, in which case re-analysis
  oriented to classification problems is infeasible regardless of format conversion.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Open-source modular platform whose DetectFormat module scans the study file
    tree, matches it against supported format signatures, and dispatches the converter
    for the custom standard format
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Distribution mechanism through which CloMet is run to detect and convert study
    formats
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:58c5ea591d2289efe14d57d77bbccb6d5670d0d35b976fd61aaaf54b3be18c90
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboLights
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# file-system-format-detection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Detect the raw file-system format of NMR metabolomics studies retrieved from MetaboLights or Metabolomics Workbench by inspecting file extensions and directory structure, then classify each study as one of CloMet's supported formats (single .fid, .mzXML, .CNX, .nmrML, or .TABULAR files, or Bruker directories) for conversion to a custom directory-based standard format.

## When to use

Use when a downloaded metabolomics study from MetaboLights or Metabolomics Workbench arrives in an unknown raw layout and it must be classified into a known format signature before transformation into an analysis-ready standard format.

## When NOT to use

- The data are already in the custom CloMet standard format or directly consumable by MetaboAnalyst or Workflows4Metabolomics — no format detection is needed.
- CloMet itself does not implement data processing algorithms, so this skill cannot be used for downstream spectral processing such as the NMR_Read, NMR_Preprocessing, or NMR_Bucketing steps.
- Class metadata is absent from the repository data set, in which case re-analysis oriented to classification problems is infeasible regardless of format conversion.

## Inputs

- Raw file-system tree of a metabolomics study retrieved from MetaboLights or Metabolomics Workbench (via a study ID)
- Single files with .fid, .mzXML, .CNX, .nmrML, or .TABULAR extensions
- Bruker-format directories containing acqu, acqus, fid files and a pdata directory

## Outputs

- Detected format classification for each study (one of the supported format signatures)
- Converted study directory in the custom standard format (study ID, a_/i_/m_/s_/audit metadata files, per-sample subdirectories with experiment directories holding acqu, acqus, fid files and a pdata directory)
- Invalid/undetected studies flagged as unconvertible

## How to apply

Scan the downloaded study's file tree and inspect file extensions and directory structure rather than file names to classify the format. Match the observed layout against 11 supported format signatures: single files carrying .fid, .mzXML, .CNX, .nmrML, or .TABULAR extensions, and Bruker directories distinguished by their internal folder layout (acqu, acqus, fid files plus a pdata directory). Dispatch each detected format to its corresponding converter routine, which emits the custom standard format in which each study is a directory with a study ID, metadata files (a_, i_, m_, s_, audit) at the top level, and per-sample subdirectories containing experiment directories with acqu, acqus, fid files and a pdata directory. Validate each converted study by checking for the presence of a 1r file; leave undetected or unconvertible studies untransformed and flag them as invalid.

## Related tools

- **CloMet** (Open-source modular platform whose DetectFormat module scans the study file tree, matches it against supported format signatures, and dispatches the converter for the custom standard format) — https://github.com/rmallol/clomet
- **Docker** (Distribution mechanism through which CloMet is run to detect and convert study formats)

## Evaluation signals

- Every converted study directory contains the 1r file expected in each sample; any study lacking the 1r file in any sample is considered invalid.
- The converted study tree shows a study ID with a_, i_, m_, s_, and audit metadata files at the top level, and per-sample subdirectories containing experiment directories with acqu, acqus, fid files and a pdata directory.
- Conversion succeeds for the expected proportion of studies — in the reported test of 49 data sets, 47 (95.9%) were successfully transformed; a large deviation on comparable data suggests detection or conversion failures.
- Studies whose format could not be detected or converted remain untransformed and are flagged as invalid rather than silently passed through.

## Limitations

- CloMet relies on the developers' community to extend and maintain it; new repository formats require community-contributed modules.
- Studies lacking the 1r file in any sample are considered invalid and cannot be converted.
- CloMet does not implement data processing algorithms; conversion only harmonizes the file-system format and does not replace analysis platforms.
- Integration with existing analysis platforms is a future plan, not currently implemented.

## Evidence
