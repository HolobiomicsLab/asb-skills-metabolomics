---
name: nmr-metabolomics-class-discrimination-roc-analysis
description: Use when re-analyzing a public NMR metabolomics study (MetaboLights or
  Metabolomics Workbench study id) to reproduce reported binary class-discrimination
  AUCs, e.g. disease vs control groups with labels such as bacterial, viral, control.
when_to_use_negative:
- Class information is not available along with the metabolomics data in the public
  repository — re-analysis oriented to classification problems won't be feasible.
- You need data processing algorithms (preprocessing beyond format conversion) — CloMet
  itself does not implement data processing; use MetaboAnalyst or Workflows4Metabolomics
  modules instead.
- Input is already a feature table directly loadable into MetaboAnalyst — no CloMet
  conversion step is needed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: CloMet
  role: Converts the raw NMR metabolomics study from MetaboLights into a standardized
    csv usable directly in MetaboAnalyst
  repo: https://github.com/rmallol/clomet
- name: MetaboAnalyst
  role: Web-based platform whose Statistical Analysis module performs the ROC/AUC
    class-discrimination evaluation
- name: Docker
  role: Runtime container via which CloMet is executed on a study id
- name: MetaboLights
  role: Repository from which the study files (e.g. MTBLS563) are retrieved by study
    identifier
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:cfe4ab01691d044bbd43e5cfb33fdc1d1b2f5980561dce0e2ba6d9f59a907631
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nmr-metabolomics-class-discrimination-roc-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reproduce published class-discrimination ROC/AUC results from public NMR-based metabolomics data sets by converting repository studies (e.g. MTBLS563 from MetaboLights) with CloMet and running ROC/AUC evaluation in MetaboAnalyst's Statistical Analysis module. This connects repository data accessibility to downstream classification re-analysis.

## When to use

Use when re-analyzing a public NMR metabolomics study (MetaboLights or Metabolomics Workbench study id) to reproduce reported binary class-discrimination AUCs, e.g. disease vs control groups with labels such as bacterial, viral, control.

## When NOT to use

- Class information is not available along with the metabolomics data in the public repository — re-analysis oriented to classification problems won't be feasible.
- You need data processing algorithms (preprocessing beyond format conversion) — CloMet itself does not implement data processing; use MetaboAnalyst or Workflows4Metabolomics modules instead.
- Input is already a feature table directly loadable into MetaboAnalyst — no CloMet conversion step is needed.

## Inputs

- MetaboLights study identifier (e.g. MTBLS563) or Metabolomics Workbench study id
- Raw NMR-based metabolomics data files downloaded from the repository
- Class/group metadata labels (e.g. bacterial, viral, control) accompanying the study
- CloMet Docker image built from github.com/rmallol/clomet

## Outputs

- Ready-to-use csv in CloMet's standardized output format with preserved sample rows and class labels
- ROC curves and AUC values per binary comparison (e.g. bacterial vs control = 0.82, viral vs control = 0.84)
- auc_results.csv recording the reproduced AUC values

## How to apply

Retrieve the study files from the repository using its study id (e.g. MTBLS563), then run the CloMet converter — available through a Docker file — on the study id to produce a ready-to-use csv in CloMet's standardized directory-based format. Inspect the converted csv to confirm that sample rows and class labels (bacterial, viral, control) were preserved, since re-analysis for classification problems is infeasible if class metadata is not available along with the metabolomics data. Upload the converted csv into MetaboAnalyst, open the Statistical Analysis module, and perform class-discrimination analysis with ROC/AUC evaluation comparing each disease group against control (bacterial vs control, viral vs control). Record the reproduced AUC values and compare them against the article's reported values (0.82 bacterial vs control, 0.84 viral vs control) as the success criterion. The rationale is that CloMet's aim is to fill the data-accessibility gap — converting the original file system format into one directly usable in MetaboAnalyst or Workflows4Metabolomics — while the statistical discrimination itself is left to the analysis platform.

## Related tools

- **CloMet** (Converts the raw NMR metabolomics study from MetaboLights into a standardized csv usable directly in MetaboAnalyst) — https://github.com/rmallol/clomet
- **MetaboAnalyst** (Web-based platform whose Statistical Analysis module performs the ROC/AUC class-discrimination evaluation)
- **Docker** (Runtime container via which CloMet is executed on a study id)
- **MetaboLights** (Repository from which the study files (e.g. MTBLS563) are retrieved by study identifier)

## Examples

```
docker run rmallol/clomet --study-id MTBLS563 --output converted_mtbls563.csv  # then upload converted_mtbls563.csv to MetaboAnalyst Statistical Analysis for ROC/AUC (bacterial vs control, viral vs control)
```

## Evaluation signals

- Reproduced ROC AUC for bacterial vs control matches the reported 0.82 (within reasonable numerical tolerance).
- Reproduced ROC AUC for viral vs control matches the reported 0.84.
- Converted csv retains all sample rows and the class labels (bacterial, viral, control) exactly as in the source study metadata.
- CloMet output is loadable into MetaboAnalyst's Statistical Analysis module without manual reformatting (ready-to-use format).
- The comparison sets are strictly binary (each disease group vs control), matching the reported discrimination design.

## Limitations

- If information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible.
- CloMet itself does not implement data processing algorithms; preprocessing and statistics are out of its scope and delegated to analysis platforms.
- Maintenance and extension of CloMet depend on the open-source developer community, and no software version number is stated in the discussion.
- Integration with existing analysis platforms remains a future plan, not implemented.

## Evidence

- [discussion] Class metadata requirement for classification re-analysis: [evidence span withheld — anchor status: not_found]
