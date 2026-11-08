---
name: metabolomics-data-standardization
description: Use when raw NMR metabolomics files acquired from MetaboLights or Metabolomics
  Workbench need conversion into an analysis-ready format for statistical analysis,
  or when reproducing a published analysis from a repository study id.
when_to_use_negative:
- The data set is already a processed feature table or standard analysis-ready format
  — CloMet targets raw NMR data whose file system format blocks reuse
- Class metadata is unavailable alongside the metabolomics data in the repository
  — re-analysis oriented to classification problems won't be feasible
- You need data processing algorithms (preprocessing, bucketing algorithms implemented
  yourself) — CloMet's scope is data accessibility/standardization, not processing
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CloMet
  role: 'Core converter: harmonizes repository file systems into an internal directory-based
    standard and produces analysis-ready output from a study id; run via its Docker
    file'
  repo: github.com/rmallol/clomet
- name: Docker
  role: Distribution/runtime container for CloMet
- name: MetaboAnalyst
  role: 'Downstream destination platform: consumes CloMet''s converted output for
    Statistical Analysis and ROC/AUC evaluation'
- name: Workflows4Metabolomics
  role: 'Alternative downstream destination: its tools (NMR_Read, NMR_Preprocessing,
    NMR_Bucketing) require the format CloMet provides'
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:396704885776135980e3b2fef41baf810a34137c8dbf6456a5efeed764a418b9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  - Workflows4Metabolomics
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolomics-data-standardization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardizes raw, NMR-based metabolomics data sets retrieved from public repositories (MetaboLights, Metabolomics Workbench) into a file format directly loadable into MetaboAnalyst or Workflows4Metabolomics, using the open-source CloMet platform. It addresses the lack of file-system standardization in repository data that blocks reuse by non-expert users.

## When to use

Use when raw NMR metabolomics files acquired from MetaboLights or Metabolomics Workbench need conversion into an analysis-ready format for statistical analysis, or when reproducing a published analysis from a repository study id.

## When NOT to use

- The data set is already a processed feature table or standard analysis-ready format — CloMet targets raw NMR data whose file system format blocks reuse
- Class metadata is unavailable alongside the metabolomics data in the repository — re-analysis oriented to classification problems won't be feasible
- You need data processing algorithms (preprocessing, bucketing algorithms implemented yourself) — CloMet's scope is data accessibility/standardization, not processing

## Inputs

- Repository study identifier (e.g. MTBLS563) from MetaboLights or Metabolomics Workbench
- Raw NMR-based metabolomics acquisition files (fid, 1r, acqu/acqus) with associated study metadata
- Private/local metabolomics data sets via the LOCAL tab

## Outputs

- Ready-to-use data set in CloMet's standardized, directory-based format
- Converted file compatible with direct upload into MetaboAnalyst or Workflows4Metabolomics
- Binned sample-by-spectral-variable matrix (e.g. 1000 binned columns via binning/bucketing)

## How to apply

Retrieve the target study by inserting its repository study id (e.g. MTBLS563) into CloMet, which is distributed as a Docker file, so a ready-to-use data set is produced with minimal interaction. CloMet harmonizes the heterogeneous file systems found in public repositories into an internal, directory-based custom data standard that preserves per-sample acquisition files (NMR fid/1r/acqu files) and class metadata. Inspect the converted output to confirm sample rows and class labels were preserved, since re-analysis for classification is infeasible if class information is missing from the repository. The standardized output can then be plugged directly into downstream analysis platforms such as MetaboAnalyst's Statistical Analysis module or Workflows4Metabolomics tools (NMR_Read, NMR_Preprocessing, NMR_Bucketing); CloMet itself deliberately implements no processing algorithms. For private data, use CloMet's LOCAL tab to translate local files through the same standard.

## Related tools

- **CloMet** (Core converter: harmonizes repository file systems into an internal directory-based standard and produces analysis-ready output from a study id; run via its Docker file) — github.com/rmallol/clomet
- **Docker** (Distribution/runtime container for CloMet)
- **MetaboAnalyst** (Downstream destination platform: consumes CloMet's converted output for Statistical Analysis and ROC/AUC evaluation)
- **Workflows4Metabolomics** (Alternative downstream destination: its tools (NMR_Read, NMR_Preprocessing, NMR_Bucketing) require the format CloMet provides)

## Evaluation signals

- The converted data set loads directly into MetaboAnalyst's Statistical Analysis module without manual reformatting
- Sample rows and class labels (e.g. bacterial, viral, control) are preserved after conversion — a missing-class output predicts classification re-analysis failure
- Downstream reproduction of published results matches reported metrics (e.g. MTBLS563 ROC AUCs of 0.82 bacterial vs control and 0.84 viral vs control)
- If binning is applied downstream, the binned matrix has exactly 1000 spectral columns as specified in the paper
- FAIR provenance is inherited: the output remains traceable to the original MetaboLights/Metabolomics Workbench study id

## Limitations

- CloMet itself does not implement data processing algorithms; data processing is out of its scope and left to other developers
- If class information is not available with the metabolomics data in the repository, re-analysis oriented to classification problems won't be feasible
- CloMet's open-source nature (on GitHub) relies on the developers' community to extend and maintain it
- Integration with existing analysis platforms is a future plan, not yet implemented; no software version number is stated for CloMet

## Evidence
