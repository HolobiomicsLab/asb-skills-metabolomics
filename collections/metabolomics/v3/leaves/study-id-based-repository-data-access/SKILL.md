---
name: study-id-based-repository-data-access
description: Use when a ready-to-use NMR metabolomics data set from MetaboLights or
  Metabolomics Workbench is needed and only the study identifier (e.g. an MTBLS or
  Metabolomics Workbench study id) is available, and the raw repository file system
  is non-standard.
when_to_use_negative:
- The user's data are private/local rather than from a public repository — CloMet's
  LOCAL tab is a separate translation path for users' own private data.
- The goal is downstream statistical analysis or data processing itself — CloMet deliberately
  does not implement data processing algorithms; use MetaboAnalyst or Workflows4Metabolomics
  for that.
- Class metadata is missing in the repository data set — re-analysis oriented to classification
  problems won't be feasible even after successful retrieval.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: platform that retrieves the repository data set by study id, detects its raw
    file-system format, and converts it to the standardized format
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: container runtime used to install and launch CloMet
- name: MetaboLights
  role: source repository whose data sets are retrieved by study id
- name: Metabolomics Workbench
  role: source repository whose data sets are retrieved by study id
- name: MetaboAnalyst
  role: downstream analysis platform consuming the converted output
- name: Workflows4Metabolomics
  role: downstream analysis platform consuming the converted output (NMR_Read, NMR_Preprocessing,
    NMR_Bucketing)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:5f3281b3fda49afd7915fd1ddc4f7a3c31c432d5bff9b2a37cf79a8e7103dc07
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  - Metabolomics Workbench
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# study-id-based-repository-data-access

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A skill for retrieving public NMR metabolomics data sets by study identifier from MetaboLights or Metabolomics Workbench via CloMet, which harmonizes heterogeneous raw file-system formats into a single directory-based standard ready for analysis platforms like MetaboAnalyst or Workflows4Metabolomics.

## When to use

Use when a ready-to-use NMR metabolomics data set from MetaboLights or Metabolomics Workbench is needed and only the study identifier (e.g. an MTBLS or Metabolomics Workbench study id) is available, and the raw repository file system is non-standard.

## When NOT to use

- The user's data are private/local rather than from a public repository — CloMet's LOCAL tab is a separate translation path for users' own private data.
- The goal is downstream statistical analysis or data processing itself — CloMet deliberately does not implement data processing algorithms; use MetaboAnalyst or Workflows4Metabolomics for that.
- Class metadata is missing in the repository data set — re-analysis oriented to classification problems won't be feasible even after successful retrieval.

## Inputs

- study identifier from MetaboLights or Metabolomics Workbench (e.g. MTBLS study id)
- public NMR-based untargeted metabolomics data set with raw acquisition files and associated metadata

## Outputs

- ready-to-use data set in CloMet's custom directory-based standard format
- converted files directly loadable in MetaboAnalyst or Workflows4Metabolomics (e.g. via NMR_Read)
- per-study transformation outcome record (success/failure) for tallied success rates

## How to apply

Install Docker and run CloMet (available through a Docker file, source at github.com/rmallol/clomet). Insert only the study id of the target study so CloMet retrieves the corresponding repository data set automatically. Let CloMet detect the raw file-system format of the acquired data set, then run its conversion, which translates the original file system into CloMet's custom directory-based internal standard that harmonizes the different formats found in public repositories. The output is a ready-to-use data set with minimal user interaction, formatted so it can be plugged directly into tools such as NMR_Read, NMR_Preprocessing, or NMR_Bucketing in Workflows4Metabolomics or into MetaboAnalyst. In validation, CloMet successfully transformed 47 of 49 (95.9%) eligible NMR data sets, so treat rare failures as an expected edge case and record per-study outcomes.

## Related tools

- **CloMet** (platform that retrieves the repository data set by study id, detects its raw file-system format, and converts it to the standardized format) — https://github.com/rmallol/clomet
- **Docker** (container runtime used to install and launch CloMet)
- **MetaboLights** (source repository whose data sets are retrieved by study id)
- **Metabolomics Workbench** (source repository whose data sets are retrieved by study id)
- **MetaboAnalyst** (downstream analysis platform consuming the converted output)
- **Workflows4Metabolomics** (downstream analysis platform consuming the converted output (NMR_Read, NMR_Preprocessing, NMR_Bucketing))

## Evaluation signals

- CloMet's format detection correctly identifies the raw file-system format of the retrieved study before conversion.
- The converted output loads directly in MetaboAnalyst or via NMR_Read/NMR_Preprocessing/NMR_Bucketing in Workflows4Metabolomics without manual restructuring.
- Across a validation batch of eligible NMR data sets, the success tally reproduces the reported rate (47/49, 95.9%); per-study outcomes recorded in transformation_results.csv match the article's counts for validated data sets.
- Retrieved data retain the FAIR properties inherited from the source repository (findability, accessibility, interoperability, reusability).
- Only a study id was required to obtain the complete ready-to-use data set (minimal interaction).

## Limitations

- Not all repository data sets transform successfully: 2 of 49 eligible NMR data sets failed conversion in the article's evaluation.
- If information about classes is not available along with the metabolomics data in the repository, re-analysis oriented to classification problems won't be feasible.
- CloMet does not implement data processing algorithms; extending it relies on the open-source developer community to maintain and extend its modular codebase.
- Integration with existing analysis platforms is a future plan, not an implemented feature.

## Evidence
