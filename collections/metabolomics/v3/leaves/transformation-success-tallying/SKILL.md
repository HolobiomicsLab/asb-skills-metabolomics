---
name: transformation-success-tallying
description: Use when batch-running CloMet over multiple MetaboLights or Metabolomics
  Workbench study IDs and you need per-study success/failure outcomes plus an aggregate
  transformation rate to reproduce or extend Figure 8A.
when_to_use_negative:
- You only need to transform a single private/local data set via CloMet's LOCAL tab
  rather than benchmark success rates over repository studies.
- The input is MS-based (GC-MS/LC-MS) metabolomics data rather than NMR-based, which
  is CloMet's stated focus.
- Class metadata is missing from the repository data sets and the goal is re-analysis
  for classification problems, not format transformation.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3174
tools:
- name: CloMet
  role: Detects each data set's raw file-system format and converts it to the analysis-ready
    format; per-study outcomes are tallied by this skill
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Runtime used to launch CloMet for each study via its Docker file
- name: MetaboLights
  role: Source repository whose study IDs are iterated over for transformation testing
- name: Metabolomics Workbench
  role: Source repository whose study IDs are iterated over for transformation testing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:485ec3067563ed0b8f9a7664055bd00ab9a508310bdb8e049e9f1e87fc32911a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboLights
  - Metabolomics Workbench
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# transformation-success-tallying

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Systematically records and tallies per-dataset transformation outcomes when running CloMet over public NMR metabolomics data sets, yielding an overall success rate comparable to the article's reported 47/49 (95.9%).

## When to use

Use when batch-running CloMet over multiple MetaboLights or Metabolomics Workbench study IDs and you need per-study success/failure outcomes plus an aggregate transformation rate to reproduce or extend Figure 8A.

## When NOT to use

- You only need to transform a single private/local data set via CloMet's LOCAL tab rather than benchmark success rates over repository studies.
- The input is MS-based (GC-MS/LC-MS) metabolomics data rather than NMR-based, which is CloMet's stated focus.
- Class metadata is missing from the repository data sets and the goal is re-analysis for classification problems, not format transformation.

## Inputs

- List of eligible NMR study IDs from MetaboLights and Metabolomics Workbench
- CloMet Docker image (github.com/rmallol/clomet)
- Raw file-system data sets retrieved per study id

## Outputs

- transformation_results.csv with per-dataset success/failure outcomes
- Overall transformation success rate (tally of successes / total data sets)

## How to apply

Enumerate the eligible NMR data sets from MetaboLights and Metabolomics Workbench that fit the selection criteria (49 in the article), then launch CloMet via its Docker file for each study by inserting the study id so the platform retrieves the repository data set. Let CloMet detect the raw file-system format of each acquired data set and run its conversion into the format usable by analysis platforms such as MetaboAnalyst or Workflows4Metabolomics. For every data set, record a binary success/failure outcome, tally the successes, and compute the overall rate (expected ballpark: ~95.9%, i.e. 47 of 49). Write the per-dataset outcomes and the tally to transformation_results.csv, and validate by checking the success count against the article's reported count for the validated data sets.

## Related tools

- **CloMet** (Detects each data set's raw file-system format and converts it to the analysis-ready format; per-study outcomes are tallied by this skill) — https://github.com/rmallol/clomet
- **Docker** (Runtime used to launch CloMet for each study via its Docker file)
- **MetaboLights** (Source repository whose study IDs are iterated over for transformation testing)
- **Metabolomics Workbench** (Source repository whose study IDs are iterated over for transformation testing)

## Examples

```
docker run rmallol/clomet --study <STUDY_ID>  # record outcome per study, then tally successes / 49 to transformation_results.csv
```

## Evaluation signals

- Number of successful transformations equals the count reported in the article for the validated data sets (47 of 49).
- Every eligible study ID has exactly one recorded outcome (success or failure) in transformation_results.csv — no missing rows.
- Computed success rate falls at or near 95.9% when using the article's selection criteria.
- Each successfully converted data set is loadable by downstream analysis platforms such as MetaboAnalyst or Workflows4Metabolomics tools (e.g. NMR_Read, NMR_Preprocessing, NMR_Bucketing).

## Limitations

- Re-analysis for classification problems is infeasible when class metadata is not available alongside the metabolomics data in the public repositories.
- CloMet does not implement data processing algorithms; extension and maintenance depend on the open-source developer community.
- No specific CloMet software version number is stated in the article, which may affect reproducibility of the tally.

## Evidence
