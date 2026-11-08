---
name: msdial-annotation-scoring
description: Use when a pre-processed MS-DIAL or patRoon untargeted metabolomics feature
  table with metabolite annotations needs one confidence level per feature, written
  out as annoLevels.csv.
when_to_use_negative:
- The input is raw spectra or unprocessed data rather than a pre-processed feature
  table exported from MS-DIAL or patRoon
- The goal is pooled-QC feature filtering rather than annotation-level assignment
  (GetFeatistics handles QC filtration as a separate function)
- The input comes from tools other than patRoon or MS-DIAL (e.g. XCMS alone), for
  which dedicated import functions are not described here
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: GetFeatistics
  role: R-package providing get_AnnoLevels_MSDial (dot-product + fragment-presence
    cut-offs) and get_feat_info_from_patRoon (MoNAScore cut-offs) to assign Schymanski
    confidence levels
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: MS-DIAL
  role: Source of untargeted feature tables whose annotations are graded by the MS-DIAL
    scoring path
- name: patRoon
  role: Source of untargeted feature tables whose annotations are graded via MoNAScore
    cut-offs; patRoon itself also includes 'Automatic estimation of confidence levels'
    natively
  repo: https://github.com/rickhelmus/patRoon
- name: R
  role: Runtime environment for the GetFeatistics package (R >= 4.3.1 plus Java with
    matching architecture)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:da07d4d1a342c549d12e31b5c5854d4167898e3ddfb58a6a134386059916c1a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- ms-dial-annotation-scoring-integration
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# msdial-annotation-scoring

## Summary

Assign Schymanski et al. annotation confidence levels to MS-DIAL and patRoon feature-table annotations inside the GetFeatistics R-package, using dot-product similarity and fragment-presence cut-offs for MS-DIAL and MoNAScore cut-offs for patRoon.

## When to use

Use when a pre-processed MS-DIAL or patRoon untargeted metabolomics feature table with metabolite annotations needs one confidence level per feature, written out as annoLevels.csv.

## When NOT to use

- The input is raw spectra or unprocessed data rather than a pre-processed feature table exported from MS-DIAL or patRoon
- The goal is pooled-QC feature filtering rather than annotation-level assignment (GetFeatistics handles QC filtration as a separate function)
- The input comes from tools other than patRoon or MS-DIAL (e.g. XCMS alone), for which dedicated import functions are not described here

## Inputs

- MS-DIAL output feature table with metabolite annotations
- patRoon output feature table with metabolite annotations

## Outputs

- annoLevels.csv with one Schymanski confidence level per feature

## How to apply

Import the pre-processed feature table (MS-DIAL or patRoon output) into the GetFeatistics R environment (R >= 4.3.1). For MS-DIAL tables, call get_AnnoLevels_MSDial, which grades each annotation against dot-product similarity cut-offs combined with fragment-presence checks, and maps the result onto the Schymanski et al. confidence levels. For patRoon tables, call get_feat_info_from_patRoon, which grades annotations using MoNAScore cut-offs mapped to the same Schymanski levels. The decision point is which importer the input table came from, since the two pipelines use different scoring metrics. Write the graded annotations — one confidence level per feature — to annoLevels.csv.

## Related tools

- **GetFeatistics** (R-package providing get_AnnoLevels_MSDial (dot-product + fragment-presence cut-offs) and get_feat_info_from_patRoon (MoNAScore cut-offs) to assign Schymanski confidence levels) — https://github.com/FrigerioGianfranco/GetFeatistics
- **MS-DIAL** (Source of untargeted feature tables whose annotations are graded by the MS-DIAL scoring path)
- **patRoon** (Source of untargeted feature tables whose annotations are graded via MoNAScore cut-offs; patRoon itself also includes 'Automatic estimation of confidence levels' natively) — https://github.com/rickhelmus/patRoon
- **R** (Runtime environment for the GetFeatistics package (R >= 4.3.1 plus Java with matching architecture))

## Examples

```
library(GetFeatistics); annoLevels <- get_AnnoLevels_MSDial(msdial_feature_table); write.csv(annoLevels, "annoLevels.csv")
```

## Evaluation signals

- annoLevels.csv exists and contains exactly one confidence level per feature from the imported table
- Each assigned level is a valid Schymanski et al. confidence level
- The correct scoring path was used: dot-product + fragment-presence for MS-DIAL inputs, MoNAScore for patRoon inputs
- Rerunning the function on the same feature table reproduces identical level assignments (deterministic cut-off grading)
- The feature identifiers in annoLevels.csv correspond 1:1 to the rows of the imported MS-DIAL or patRoon feature table

## Limitations

- The described import functions are specific to patRoon and MS-DIAL feature tables; other tools (e.g. raw XCMS output) are not covered by this scoring path
- Feature extraction and annotation must already be done — confidence-level assignment presumes pre-processed, annotated feature tables

## Evidence

- [abstract] For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels"
- [other] GetFeatistics automatically assigns annotation confidence levels implementing MoNAScore cut-offs for patRoon, and dot product and fragment presence cut-offs for MS-DIAL.: [evidence span withheld — anchor status: not_found]
- [other] For MS-DIAL tables, call get_AnnoLevels_MSDial and grade each annotation using dot-product similarity cut-offs combined with fragment-presence checks, mapping results onto Schymanski et al. confidence levels.: [evidence span withheld — anchor status: not_found]
- [other] Write the graded annotations, one confidence level per feature, to annoLevels.csv.: [evidence span withheld — anchor status: not_found]
- [readme] ID confidence          | Automatic estimation of confidence levels.                               | Native: [evidence span withheld — anchor status: external_doc]
