---
name: batch-metabolomics-data-format-conversion
description: Use when a batch of public NMR metabolomics studies (MetaboLights or
  Metabolomics Workbench study IDs) must be converted from their raw repository file-system
  format into an analysis-ready format for MetaboAnalyst or Workflows4Metabolomics.
when_to_use_negative:
- The data set is already in a format directly usable by MetaboAnalyst or Workflows4Metabolomics
  — conversion is unnecessary
- The goal is downstream data processing or statistical analysis itself — CloMet explicitly
  does not implement data processing algorithms, which are left to other developers
- The intended re-analysis is a classification problem but class metadata is unavailable
  alongside the public metabolomics data — re-analysis will not be feasible regardless
  of format conversion
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: 'Core conversion platform: retrieves repository data sets by study ID, detects
    the raw file-system format, and transforms it into the standardized analysis-ready
    format'
  repo: github.com/rmallol/clomet
- name: Docker
  role: Deployment mechanism through which CloMet is launched for each conversion
    run
- name: MetaboLights
  role: Source repository from which NMR data sets are retrieved by study ID
- name: Metabolomics Workbench
  role: Source repository from which NMR data sets are retrieved by study ID
- name: MetaboAnalyst
  role: Target analysis platform whose required input format defines the conversion
    output
- name: Workflows4Metabolomics
  role: Target analysis platform (NMR_Read, NMR_Preprocessing, NMR_Bucketing) whose
    required input format defines the conversion output
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:f219441035ef4fd0fc80088bfdd7c53fb0a997ac78404fbb947478e26b010789
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  - Metabolomics Workbench
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# batch-metabolomics-data-format-conversion

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Batch transformation of raw NMR-based metabolomics data sets retrieved from MetaboLights and Metabolomics Workbench into the standardized file format required by analysis platforms such as MetaboAnalyst and Workflows4Metabolomics, using the CloMet platform run through Docker. It addresses the lack of standardization in repository file-system formats that blocks data reuse.

## When to use

Use when a batch of public NMR metabolomics studies (MetaboLights or Metabolomics Workbench study IDs) must be converted from their raw repository file-system format into an analysis-ready format for MetaboAnalyst or Workflows4Metabolomics.

## When NOT to use

- The data set is already in a format directly usable by MetaboAnalyst or Workflows4Metabolomics — conversion is unnecessary
- The goal is downstream data processing or statistical analysis itself — CloMet explicitly does not implement data processing algorithms, which are left to other developers
- The intended re-analysis is a classification problem but class metadata is unavailable alongside the public metabolomics data — re-analysis will not be feasible regardless of format conversion

## Inputs

- Study identifiers for NMR-based metabolomics data sets deposited in MetaboLights or Metabolomics Workbench
- Raw NMR data sets in their original repository file-system formats (retrieved by CloMet via study ID)
- Docker environment with the CloMet image/file

## Outputs

- Converted data sets in CloMet's standardized directory-based format, directly loadable by MetaboAnalyst or Workflows4Metabolomics (e.g., NMR_Read/NMR_Preprocessing/NMR_Bucketing)
- Per-dataset transformation outcomes (success/failure) and overall success rate, e.g., 47/49 (95.9%)
- transformation_results.csv tally of batch outcomes

## How to apply

For each eligible study, launch CloMet (available as a Docker file from github.com/rmallol/clomet) and insert the study's repository ID so the platform retrieves the data set from MetaboLights or Metabolomics Workbench. Let CloMet automatically detect the raw file-system format of each acquired data set, then run its conversion step, which harmonizes the heterogeneous repository layouts into CloMet's custom directory-based internal standard that feeds directly into analysis platforms (e.g., NMR_Read, NMR_Preprocessing, NMR_Bucketing in Workflows4Metabolomics, or MetaboAnalyst). Record the per-dataset outcome (success/failure) and tally the overall transformation rate; in the reference evaluation, 47 of 49 selected NMR data sets (95.9%) were successfully transformed. Write the tallied outcomes to transformation_results.csv.

## Related tools

- **CloMet** (Core conversion platform: retrieves repository data sets by study ID, detects the raw file-system format, and transforms it into the standardized analysis-ready format) — github.com/rmallol/clomet
- **Docker** (Deployment mechanism through which CloMet is launched for each conversion run)
- **MetaboLights** (Source repository from which NMR data sets are retrieved by study ID)
- **Metabolomics Workbench** (Source repository from which NMR data sets are retrieved by study ID)
- **MetaboAnalyst** (Target analysis platform whose required input format defines the conversion output)
- **Workflows4Metabolomics** (Target analysis platform (NMR_Read, NMR_Preprocessing, NMR_Bucketing) whose required input format defines the conversion output)

## Evaluation signals

- Per-dataset success/failure outcomes recorded for every study ID processed in the batch
- Tallied success rate matches the article's benchmark of 47/49 (95.9%) successfully transformed NMR data sets
- Converted output loads directly into MetaboAnalyst or Workflows4Metabolomics tools such as NMR_Read without manual reformatting
- The number of successful transformations in the output table matches the success count reported in the article for validated data sets
- Outcomes are written to transformation_results.csv with one row per data set

## Limitations

- CloMet itself does not implement data processing algorithms; extending it with processing is out of scope and left to the developer community
- Maintenance and extension of CloMet rely on the open-source developer community
- Integration with existing analysis platforms remains a future plan, not an implemented feature
- If class metadata is not available alongside the data in public repositories, re-analysis oriented to classification problems will not be feasible even after successful conversion
- No software version number is stated for CloMet in the article

## Evidence

- [abstract] CloMet converts repository NMR data to analysis-ready format: "converts raw and NMR-based metabolomics data from MetaboLights and Metabolomics Workbench to a file format that can be used directly either in MetaboAnalyst or in Workflows4Metabolomics"
- [discussion] Study-ID-based retrieval with minimal interaction: "by just inserting a study id our users can get a ready-to-use data set with the minimal interaction possible"
- [discussion] Custom format harmonizes repository formats: "Our custom data format is an internal standard that harmonizes the different formats found in public repositories."
- [discussion] Motivation: public data sets not reusable due to format: "many public data sets are not reusable because of the format they have been shared"
- [readme] README description of purpose and NMR focus: "CloMet eases the connection between public data repositories and data analysis platforms by harmonizing the file systems of available data sets, with a focus on NMR-based metabolomics."
- [abstract] CloMet available through a Docker file: "CloMet, which is available through a Docker file, converts raw and NMR-based metabolomics data"
