---
name: processor-pipeline-architecture-design
description: Use when raw LC–MS mzML files must be curated into a quality-controlled
  feature matrix and you need a reusable, versioned sequence of filtering/correction
  processors rather than ad-hoc scripts.
when_to_use_negative:
- 'Feature detection/annotation, not matrix curation: TidyMS is explicitly noted as
  lacking feature annotation capability, so this skill is wrong for annotating putative
  metabolites'
- 'Peak picking itself: if the input is still raw vendor data and the goal is only
  retention-time alignment, deisotoping, or grouping adducts, use the extraction stage
  rather than building the curation Pipeline'
- 'Datasets without quality-control samples (pooled/intrastudy QCs, blanks): most
  processors (RSD, D-ratio, blank, prevalence) cannot be computed and the pipeline
  should not be assembled'
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3372
tools:
- name: TidyMS
  role: Provides the DataContainer, Processor, Pipeline, Filter and Corrector abstractions
    this skill encodes; executes the stored processor list
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Used to read raw mzML files into MSData objects feeding the DataContainer
- name: Pandas
  role: Holds the extracted data matrix and feature/sample descriptors inside the
    DataContainer
- name: Numpy
  role: Numerical backing for matrix operations performed by Processors
- name: Scipy
  role: Scientific computing support for TidyMS processor computations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:c7eb5c5b38c44bcd6c388d8903f564fd33fab05b69358ead1072d5536a98e759
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - Pandas
  - Scipy
  - TidyMS
  - pyopenms
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# processor-pipeline-architecture-design

## Summary

Architectural skill derived from TidyMS: model an LC–MS data-curation strategy as a fixed Pipeline of Processor objects (feature Filters and Correctors) that operate on a DataContainer holding the extracted feature matrix and its sample/feature descriptors as Pandas DataFrames.

## When to use

Use when raw LC–MS mzML files must be curated into a quality-controlled feature matrix and you need a reusable, versioned sequence of filtering/correction processors rather than ad-hoc scripts.

## When NOT to use

- Feature detection/annotation, not matrix curation: TidyMS is explicitly noted as lacking feature annotation capability, so this skill is wrong for annotating putative metabolites
- Peak picking itself: if the input is still raw vendor data and the goal is only retention-time alignment, deisotoping, or grouping adducts, use the extraction stage rather than building the curation Pipeline
- Datasets without quality-control samples (pooled/intrastudy QCs, blanks): most processors (RSD, D-ratio, blank, prevalence) cannot be computed and the pipeline should not be assembled

## Inputs

- Raw LC–MS mzML files (e.g. MetaboLights MTBLS1919 study files from NIST candidate RM 8231 and SRM 1950 plasma analyses)
- Sample metadata / run-order sheet mapping sample types (study samples, pooled/intrastudy QC, solvent blanks, zero-volume injections, process blanks) to sample classes

## Outputs

- A serialized processed DataContainer: curated data matrix with retained features and their peak areas, plus preserved per-feature (m/z, retention time) and per-sample descriptors in Pandas DataFrames
- Ordering/definition of the Processor list (filter and corrector steps) that constitutes the reusable data-curation strategy

## How to apply

Read the raw mzML with pyopenms into TidyMS MSData objects, then run TidyMS feature detection (CWT-based peak picking, chromatogram building, centWord detection) to produce the extracted data matrix. Wrap the matrix plus its per-feature and per-sample descriptors in a DataContainer whose sample-mapping attribute links sample types to sample classes so downstream filters can behave class-aware. Define a Pipeline that stores an ordered list of Processor objects — in the article's data-curation pipeline these were an Rt filter dropping features below 90 s (system dead time ≈0.8 min), a pooled/intrastudy-QC-presence filter as a precondition for LOESS batch correction, a blank filter zeroing features ≤10-fold the maximum solvent/zero-volume/process-blank area, an RSD filter cutting features with RSD >20% in pooled QCs, a 100% intraclass prevalence filter with area threshold 5, and a D-ratio filter removing features whose QC MAD is ≥10% of the all-sample MAD. Execute the Pipeline on the DataContainer and serialize the processed DataContainer to disk; the fixed architecture keeps the curation strategy independent of any single run.

## Related tools

- **TidyMS** (Provides the DataContainer, Processor, Pipeline, Filter and Corrector abstractions this skill encodes; executes the stored processor list) — https://github.com/griquelme/tidyms
- **pyopenms** (Used to read raw mzML files into MSData objects feeding the DataContainer)
- **Pandas** (Holds the extracted data matrix and feature/sample descriptors inside the DataContainer)
- **Numpy** (Numerical backing for matrix operations performed by Processors)
- **Scipy** (Scientific computing support for TidyMS processor computations)

## Examples

```
from tidyms.core import MSData; msdata = MSData.from_mzml('sample.mzml'); container = MSData.as_data_container(msdata); container = pipeline(container); container.serialize('processed.h5')
```

## Evaluation signals

- The Pipeline executes its stored Processor list over the DataContainer and yields a serialized processed DataContainer file as task output
- Per-feature descriptors (retention time, m/z) and per-sample descriptors are preserved in Pandas DataFrames of the output DataContainer
- Features with retention time <90 s (below ≈0.8 min system dead time) are absent from the curated matrix
- No retained feature has pooled-QC RSD >20% or a D-ratio ≥10% of the study-sample MAD; features ≤10-fold max blank area are set to zero
- Sample-mapping attribute correctly associates sample types to sample classes so class-aware filters behave as specified

## Limitations

- No feature annotation capability — a curated matrix still lacks putative metabolite identities
- Running time scales with sample count: 31 raw files took 66 s with ~2 s per sample for feature detection, so large studies need planning
- The article does not state software version numbers for TidyMS, Python, or scikit-learn, so exact API compatibility must be checked against the repo
- Thumbnails/thresholds (e.g. 20% RSD, area threshold 5, 10-fold blank rule, 10% D-ratio) were fit for the NIST RM 8231/SRM 1950 plasma study and may need re-tuning for other matrices

## Evidence

- [supplementary] a Pipeline object that stores a list of Processor objects encapsulating feature-filtering methods, so the sequence of Processors defines a reusable data-curation strategy independent of any run: [evidence span withheld — anchor status: not_found]
- [supplementary] Instantiate a DataContainer that stores the extracted data matrix and associated feature and sample descriptors in Pandas DataFrames: [evidence span withheld — anchor status: not_found]
- [intro] Preprocessing LC–MS-based untargeted metabolomics data involves as well the removal of unwanted features (retention time, m/z pairs) to retain only those analytically robust enough for data analysis: "Preprocessing LC–MS-based untargeted metabolomics data involves as well the removal of unwanted features (retention time, m/z pairs) to retain only those analytically robust enough"
- [results] The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
