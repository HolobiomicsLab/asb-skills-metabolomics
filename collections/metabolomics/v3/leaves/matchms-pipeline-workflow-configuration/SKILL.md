---
name: matchms-pipeline-workflow-configuration
description: Use when a mass spectral library cleaning run in matchms must be logged,
  rerun unchanged, or audited — e.g. after running SpectrumProcessor over an .msp
  library and needing its exact filter configuration preserved.
when_to_use_negative:
- The task is one-off exploratory metadata or peak filtering with no need for rerun
  reproducibility — the YAML logging adds artifacts you will not reuse.
- 'The pipeline is not matchms-based: other tools (MZMine, MSDial, OpenMS) handle
  peak picking but do not emit matchms-style YAML workflow and processing-report artifacts.'
- You need fragment-level plausibility checks (fragments matching the annotation)
  — the article notes these are only possible future expansions, so the YAML config
  cannot capture them.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package whose SpectrumProcessor/Pipeline class runs the cleaning and
    automatically emits the YAML workflow file and processing report
  repo: https://github.com/matchms/matchms
- name: Python
  role: Runtime environment for invoking matchms pipeline execution and artifact generation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:34ba4479f89786aa40bbe6f72fe83a115e21e16666ae16a5c7b992dad0f3d93c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# matchms-pipeline-workflow-configuration

## Summary

Leverages matchms' automatic emission of a YAML workflow file (recording all pipeline settings and filters) and a processing report (summarizing per-filter effects on spectra) so every MS/MS library cleaning run — e.g. on the GNPS public library — is reproducible and auditable.

## When to use

Use when a mass spectral library cleaning run in matchms must be logged, rerun unchanged, or audited — e.g. after running SpectrumProcessor over an .msp library and needing its exact filter configuration preserved.

## When NOT to use

- The task is one-off exploratory metadata or peak filtering with no need for rerun reproducibility — the YAML logging adds artifacts you will not reuse.
- The pipeline is not matchms-based: other tools (MZMine, MSDial, OpenMS) handle peak picking but do not emit matchms-style YAML workflow and processing-report artifacts.
- You need fragment-level plausibility checks (fragments matching the annotation) — the article notes these are only possible future expansions, so the YAML config cannot capture them.

## Inputs

- MS/MS spectral library file (e.g. the GNPS public mass spectral library in .msp format, 500,569 spectra)
- matchms SpectrumProcessor / Pipeline configuration (library cleaning filter set: metadata harmonization, metadata derivation, annotation repair, annotation-requirement filters)

## Outputs

- YAML workflow file containing all settings and filters used, reusable to rerun the cleaning pipeline unchanged
- Processing report giving per-filter counts of spectra removed, metadata-changed, and peak-information-changed

## How to apply

Run the matchms cleaning pipeline (metadata harmonization, derivation of missing metadata, annotation repair, and annotation-requirement filters) over the spectral library using matchms 0.26.4; the run itself automatically writes a YAML file containing all settings and filters used, which can be directly reused to rerun the identical cleaning. In the same run, matchms generates a processing report giving an overview of the number of spectra changed by each filter, differentiating between completely removing spectra, changing metadata, and changing peak information. Store the two artifacts (e.g. as workflow.yaml and processing_report.csv) and verify reproducibility by re-running the pipeline from the YAML file and confirming identical filter application. The matchms Pipeline class (matchms >= 0.18.0) supports creating, importing, exporting, and modifying such YAML workflow files, making the configuration portable and versionable.

## Related tools

- **matchms** (Python package whose SpectrumProcessor/Pipeline class runs the cleaning and automatically emits the YAML workflow file and processing report) — https://github.com/matchms/matchms
- **Python** (Runtime environment for invoking matchms pipeline execution and artifact generation)

## Examples

```
from matchms import Pipeline; pipeline = Pipeline("workflow.yaml"); pipeline.run()
```

## Evaluation signals

- workflow.yaml parses and, when used to re-instantiate the pipeline, reproduces the same cleaning result on the same input library (e.g. identical final spectrum count, 448,485 curated spectra for the 21-08-2023 GNPS download).
- processing_report.csv contains one entry per filter with counts of spectra removed, metadata-changed, and peak-changed; per-filter counts sum consistently against the input library size (500,569 GNPS spectra).
- The YAML enumerates every filter actually applied (e.g. Derive Annotation From Compound Name, Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Require Valid Annotation) with no undocumented filters.
- Re-running the pipeline from the YAML alone, without the original script, yields identical processing-report statistics.

## Limitations

- Processing report examples are only provided as supplementary tables (Table S1 and Table S2); Supplementary file 1 appears without content in the article, so template verification is limited.
- The YAML records the filter configuration but cannot encode checks that do not exist yet — e.g. fragment-matching filters or plausibility checks combining metadata and measured fragments are unimplemented future work.
- Instrument type and collision energies are not standardized by current matchms filters, so the emitted workflow config will not capture cleaning of those metadata fields.

## Evidence

- [methods] YAML workflow file creation for reproducibility: "it automatically creates a YAML file containing all settings and filters used."
- [methods] Processing report contents: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [results] GNPS run emits both artifacts: "The new library cleaning pipeline was run on the GNPS public mass spectral library. A YAML file and processing report were automatically generated."
- [readme] Pipeline class YAML support: [evidence span withheld — anchor status: external_doc]
