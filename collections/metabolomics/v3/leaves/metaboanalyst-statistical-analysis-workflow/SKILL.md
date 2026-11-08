---
name: metaboanalyst-statistical-analysis-workflow
description: Use when a public NMR/untargeted metabolomics data set (e.g., a MetaboLights
  study id) has been converted to an analysis-ready csv with class labels, and the
  goal is classification/discrimination analysis with ROC-AUC comparison between defined
  groups.
when_to_use_negative:
- Class information is not available alongside the metabolomics data in the public
  repository — re-analysis oriented to classification problems won't be feasible.
- You need raw spectral (pre-)processing algorithms (e.g., NMR_Preprocessing, NMR_Bucketing-style
  steps) — CloMet does not implement data processing, and that is out of its scope.
- The data are already in a format directly usable in MetaboAnalyst or Workflows4Metabolomics
  — no CloMet conversion step is needed.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: MetaboAnalyst
  role: Web-based platform whose Statistical Analysis module performs the class-discrimination
    ROC/AUC evaluation on the converted csv.
- name: CloMet
  role: Docker-delivered converter that turns a MetaboLights/Metabolomics Workbench
    study id into a ready-to-use csv loadable into MetaboAnalyst.
  repo: github.com/rmallol/clomet
- name: Docker
  role: Container runtime through which CloMet is made available and run on the study
    id.
- name: MetaboLights
  role: Source repository from which the study data (e.g., MTBLS563) are retrieved
    using the study identifier.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:8502a4db5fcb6ecc21b8e730220fca193b2d269063d2cb879537bafbe6e64cb7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metaboanalyst-statistical-analysis-workflow

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A statistical-analysis skill in which CloMet-converted metabolomics repository data (e.g., MetaboLights study MTBLS563 as a ready-to-use csv) is loaded into MetaboAnalyst's Statistical Analysis module and evaluated with class-discrimination ROC/AUC, reproducing published AUCs such as 0.82 (bacterial vs control) and 0.84 (viral vs control).

## When to use

Use when a public NMR/untargeted metabolomics data set (e.g., a MetaboLights study id) has been converted to an analysis-ready csv with class labels, and the goal is classification/discrimination analysis with ROC-AUC comparison between defined groups.

## When NOT to use

- Class information is not available alongside the metabolomics data in the public repository — re-analysis oriented to classification problems won't be feasible.
- You need raw spectral (pre-)processing algorithms (e.g., NMR_Preprocessing, NMR_Bucketing-style steps) — CloMet does not implement data processing, and that is out of its scope.
- The data are already in a format directly usable in MetaboAnalyst or Workflows4Metabolomics — no CloMet conversion step is needed.

## Inputs

- CloMet-converted csv from a MetaboLights or Metabolomics Workbench study (produced from a study id, e.g., MTBLS563)
- Class labels accompanying the metabolomics data (e.g., bacterial, viral, control)

## Outputs

- ROC/AUC values per class contrast (e.g., 0.82 bacterial vs control, 0.84 viral vs control)
- auc_results.csv recording the reproduced discrimination metrics

## How to apply

First ensure the repository data are in a format MetaboAnalyst can ingest directly: run CloMet (available through a Docker file) on the study identifier so that 'just inserting a study id' yields a ready-to-use data set, then inspect the converted csv to confirm sample rows and class labels (e.g., bacterial, viral, control) survived conversion. Upload the csv into MetaboAnalyst and open the Statistical Analysis module, where the acquired spectral data are statistically analyzed per the metabolomics workflow described in the paper. Perform class-discrimination analysis with ROC/AUC evaluation for each pairwise group contrast (bacterial vs control, viral vs control). Record the reproduced AUC values to a results file (e.g., auc_results.csv) and compare them against the published targets — here 0.82 and 0.84. Success is judged by the reproduced ROC AUCs matching the article's reported discrimination results; note that CloMet itself performs no statistics, so MetaboAnalyst is the analysis endpoint while CloMet only fills the data-accessibility gap.

## Related tools

- **MetaboAnalyst** (Web-based platform whose Statistical Analysis module performs the class-discrimination ROC/AUC evaluation on the converted csv.)
- **CloMet** (Docker-delivered converter that turns a MetaboLights/Metabolomics Workbench study id into a ready-to-use csv loadable into MetaboAnalyst.) — github.com/rmallol/clomet
- **Docker** (Container runtime through which CloMet is made available and run on the study id.)
- **MetaboLights** (Source repository from which the study data (e.g., MTBLS563) are retrieved using the study identifier.)

## Evaluation signals

- Reproduced ROC AUC matches the published value for each contrast: 0.82 for bacterial vs control and 0.84 for viral vs control.
- Converted csv preserves sample rows and class labels (bacterial, viral, control) exactly as in the source repository study.
- The uploaded file is accepted by MetaboAnalyst's Statistical Analysis module without manual reformatting, confirming CloMet's output format is directly usable.
- auc_results.csv exists and records one AUC per tested class contrast.
- Data lineage is traceable to a valid MetaboLights/Metabolomics Workbench study id, preserving the FAIR properties CloMet inherits.

## Limitations

- If information about classes is not available along with the metabolomics data in public repositories, classification re-analysis is infeasible.
- CloMet does not implement data processing algorithms; statistical and (pre-)processing steps must be done in external platforms.
- Maintenance and extension of CloMet depend on the open-source developer community, and integration with existing analysis platforms remains a future plan.
- AUC values are contrast-specific; only the reported bacterial-vs-control (0.82) and viral-vs-control (0.84) comparisons were validated in this use case.

## Evidence

- [discussion] classification infeasible without class metadata: [evidence span withheld — anchor status: not_found]
- [other] reproduced AUCs from task card: [evidence span withheld — anchor status: not_found]
