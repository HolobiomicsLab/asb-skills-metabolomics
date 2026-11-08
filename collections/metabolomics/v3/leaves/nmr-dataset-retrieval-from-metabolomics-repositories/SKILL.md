---
name: nmr-dataset-retrieval-from-metabolomics-repositories
description: Use when a task requires acquiring public NMR metabolomics data sets
  from MetaboLights or Metabolomics Workbench, or when raw repository file-system
  formats must be converted into analysis-ready formats for MetaboAnalyst or Workflows4Metabolomics.
when_to_use_negative:
- The data are the user's own private/local data rather than a public repository study
  — CloMet exposes a separate LOCAL tab for translating private data, so study-id
  retrieval is the wrong path
- The goal is downstream statistical analysis or data processing (e.g., classification
  re-analysis) — CloMet only harmonizes formats and explicitly does not implement
  data processing algorithms
- Class metadata needed for classification re-analysis is absent from the repository
  study, making re-analysis infeasible regardless of retrieval
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Platform that retrieves a study by its id from MetaboLights/Metabolomics Workbench,
    detects the raw file-system format, and converts it to the standard format
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Runtime container in which CloMet is launched, as CloMet is available through
    a Docker file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:8ceba56119d970639f22cc0f04c577f30c80d2ca5078e9894c7c4e601451ebaa
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboLights
  - Metabolomics Workbench
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nmr-dataset-retrieval-from-metabolomics-repositories

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves raw NMR-based metabolomics data sets from MetaboLights and Metabolomics Workbench by study id using CloMet (run via Docker), which detects each study's raw file-system layout and harmonizes it into a format directly usable by analysis platforms such as MetaboAnalyst or Workflows4Metabolomics.

## When to use

Use when a task requires acquiring public NMR metabolomics data sets from MetaboLights or Metabolomics Workbench, or when raw repository file-system formats must be converted into analysis-ready formats for MetaboAnalyst or Workflows4Metabolomics.

## When NOT to use

- The data are the user's own private/local data rather than a public repository study — CloMet exposes a separate LOCAL tab for translating private data, so study-id retrieval is the wrong path
- The goal is downstream statistical analysis or data processing (e.g., classification re-analysis) — CloMet only harmonizes formats and explicitly does not implement data processing algorithms
- Class metadata needed for classification re-analysis is absent from the repository study, making re-analysis infeasible regardless of retrieval

## Inputs

- MetaboLights or Metabolomics Workbench study id (public NMR-based untargeted metabolomics data set)
- Docker environment with the CloMet image

## Outputs

- Harmonized, ready-to-use NMR data set in a format directly loadable by MetaboAnalyst or Workflows4Metabolomics
- Per-dataset transformation outcome records (e.g., transformation_results.csv with success/failure tallies)

## How to apply

Launch CloMet, which is available through a Docker file, and insert the target study's id so the platform retrieves the data set from MetaboLights or Metabolomics Workbench. Let CloMet detect the raw file-system format in which the acquisition files are stored, then run its conversion, which harmonizes the heterogeneous repository layouts into the custom directory-based internal standard usable directly by MetaboAnalyst or Workflows4Metabolomics tools such as NMR_Read, NMR_Preprocessing, and NMR_Bucketing. Record per-study outcomes (success/failure) and tally the overall transformation rate — in the article's evaluation, 47 of 49 selected NMR data sets (95.9%) were successfully transformed. Rationale: raw data in these repositories lack standardization in file-system format, and study-id-based retrieval minimizes user interaction while inheriting the repositories' FAIR principles.

## Related tools

- **CloMet** (Platform that retrieves a study by its id from MetaboLights/Metabolomics Workbench, detects the raw file-system format, and converts it to the standard format) — https://github.com/rmallol/clomet
- **Docker** (Runtime container in which CloMet is launched, as CloMet is available through a Docker file)

## Evaluation signals

- The converted data set loads directly in MetaboAnalyst or in Workflows4Metabolomics tools (NMR_Read, NMR_Preprocessing, NMR_Bucketing) without manual format repair
- The count of successful transformations matches reported validation counts (e.g., 47/49 = 95.9% success across the article's 49 NMR data sets)
- Each retrieved study's harmonized directory layout matches CloMet's custom internal-standard format that reconciles MetaboLights and Metabolomics Workbench layouts
- Per-dataset success/failure outcomes are logged and tallied in the results file (transformation_results.csv)
- Studies with missing class metadata are flagged, since re-analysis for classification is then infeasible

## Limitations

- Raw data in MetaboLights and Metabolomics Workbench lack standardized file-system formats, so some studies may fail detection/conversion (2 of 49 failed in the article's test)
- CloMet does not implement data processing algorithms; extending it with processing is left to other developers
- Maintenance and extension of CloMet rely on the open-source developer community
- CloMet's aim is to fill the data-accessibility gap, not to compete with or replace existing analysis platforms; integration with them remains a future plan

## Evidence
