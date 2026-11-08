---
name: yaml-workflow-reproducibility-archiving
description: Use when running a matchms SpectrumProcessor/Pipeline cleaning job (e.g.
  on an MS/MS library such as the GNPS public library) and you must guarantee reproducibility
  and transparency of every filter applied.
when_to_use_negative:
- The task is one-off ad-hoc spectrum filtering where no rerun or audit trail is required
  — the YAML/report emission is aimed at reproducible, repeatable library-cleaning
  workflows
- The pipeline is built outside the matchms framework (e.g. MZMine, MSDial, OpenMS
  modules); the automatic YAML/report generation is a matchms Pipeline/SpectrumProcessor
  feature
- You need filters checking that measured fragments match the annotation — the article
  lists fragment-matching filters only as future expansion, so the archived YAML/report
  will not capture that validation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package whose Pipeline/SpectrumProcessor automatically emits the YAML
    workflow file and processing report during a cleaning run
  repo: https://github.com/matchms/matchms
- name: matchms 0.26.4
  role: Specific version used to run the GNPS library cleaning pipelines that generated
    the YAML file and processing report
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:f5a8891cb68dda25ec85891349516845398e38e59b4b37d184a7ef14a672f884
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  - matchms 0.26.4
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# yaml-workflow-reproducibility-archiving

## Summary

Captures a matchms spectrum-cleaning run as two automatically emitted logging artifacts — a YAML file recording every setting and filter used, plus a processing report CSV summarizing per-filter effects — so the pipeline can be rerun unchanged and each cleaning step audited.

## When to use

Use when running a matchms SpectrumProcessor/Pipeline cleaning job (e.g. on an MS/MS library such as the GNPS public library) and you must guarantee reproducibility and transparency of every filter applied.

## When NOT to use

- The task is one-off ad-hoc spectrum filtering where no rerun or audit trail is required — the YAML/report emission is aimed at reproducible, repeatable library-cleaning workflows
- The pipeline is built outside the matchms framework (e.g. MZMine, MSDial, OpenMS modules); the automatic YAML/report generation is a matchms Pipeline/SpectrumProcessor feature
- You need filters checking that measured fragments match the annotation — the article lists fragment-matching filters only as future expansion, so the archived YAML/report will not capture that validation

## Inputs

- Input mass spectral library (e.g. msp/MGF export of the GNPS public mass spectral library, downloaded 21-08-2023)
- matchms SpectrumProcessor / Pipeline configuration with the chosen filter set (matchms 0.26.4)

## Outputs

- workflow.yaml — YAML file containing all pipeline settings and filters used, directly reusable to rerun the cleaning pipeline
- processing_report.csv — per-filter overview of the number of spectra removed, with changed metadata, or with changed peak information

## How to apply

Build the processing pipeline with matchms's SpectrumProcessor/Pipeline class (version 0.26.4 in the paper) using the desired filter set — e.g. the library cleaning configuration with metadata harmonization, derivation of missing metadata (SMILES/InChI/InChIKey via RDKit and PubChem), annotation repair, and annotation-requirement filters. Run the pipeline over all spectra; matchms automatically creates a YAML file containing all settings and filters used, which the Pipeline class can import, export, or modify via yaml files and which can be directly reused to rerun the cleaning pipeline. In the same run, a processing report is generated that counts, per filter, how many spectra were completely removed, had metadata changed, or had peak information changed. Store the artifacts as workflow.yaml and processing_report.csv and verify the YAML can rerun the pipeline unchanged. The paper's reference run applied this to 500,569 GNPS library spectra (6 h 45 min runtime).

## Related tools

- **matchms** (Python package whose Pipeline/SpectrumProcessor automatically emits the YAML workflow file and processing report during a cleaning run) — https://github.com/matchms/matchms
- **matchms 0.26.4** (Specific version used to run the GNPS library cleaning pipelines that generated the YAML file and processing report) — https://github.com/matchms/matchms

## Examples

```
from matchms import Pipeline; pipeline = Pipeline("workflow.yaml"); pipeline.run()  # rerun archived cleaning pipeline unchanged
```

## Evaluation signals

- The saved workflow.yaml can be loaded back by the matchms Pipeline class and rerun unchanged, reproducing the same cleaned library
- processing_report.csv contains one row per filter with counts of spectra completely removed, metadata changed, and peak information changed
- YAML file enumerates the full filter set (e.g. Derive Annotation From Compound Name, Repair Adduct And Parent Mass Based On Smiles, Require Valid Annotation) with settings matching the run configuration
- Rerunning the same YAML on the same 500,569-spectrum GNPS input yields consistent per-filter counts in the report (reference runtime: 6 h 45 min)

## Limitations

- Processing report examples are only referenced as supplementary tables (Table S1 and Table S2); Supplementary file 1 appears without content in the document
- Reproducibility via the YAML is tied to the matchms version used (0.26.4 in the paper); behavior of filters may change across versions
- The archived YAML/report do not include plausibility checks considering both metadata and measured fragments — wrong chemical annotations consistent with the measured mass go unnoticed

## Evidence

- [methods] YAML workflow file creation for reproducibility: "it automatically creates a YAML file containing all settings and filters used."
- [methods] processing report generation: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [readme] Pipeline class yaml support: [evidence span withheld — anchor status: external_doc]
- [results] GNPS demonstration run: "The new library cleaning pipeline was run on the GNPS public mass spectral library. A YAML file and processing report were automatically generated."
