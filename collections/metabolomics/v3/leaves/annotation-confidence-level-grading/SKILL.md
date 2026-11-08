---
name: annotation-confidence-level-grading
description: Use when you have a pre-processed untargeted feature table from patRoon
  or MS-DIAL and need one Schymanski confidence level per annotated feature before
  statistics or reporting.
when_to_use_negative:
- The study is targeted and quantification is calibration-curve based — GetFeatistics
  handles that via a separate targeted workflow, not annotation-level grading
- The feature table was produced by a tool other than patRoon or MS-DIAL (e.g. raw
  XCMS output), since the dedicated get_AnnoLevels_MSDial / get_feat_info_from_patRoon
  functions expect those two formats
- Features have no annotations at all — confidence-level grading operates on existing
  annotations, not on blank/QC filtering
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GetFeatistics
  role: R package whose get_AnnoLevels_MSDial and get_feat_info_from_patRoon functions
    grade annotations onto Schymanski confidence levels and write annoLevels.csv
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Source of imported feature tables whose annotations are graded with MoNAScore
    cut-offs; patRoon itself also offers automatic estimation of confidence levels
    natively
  repo: https://github.com/rickhelmus/patRoon
- name: MS-DIAL
  role: Source of imported feature tables whose annotations are graded with dot-product
    similarity and fragment-presence cut-offs
- name: R
  role: Runtime environment for GetFeatistics (version ≥ 4.3.1 required, with matching-architecture
    Java)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:e33340d4ef722245abe9204b77d7c33f465bae3cd3a2c5956ee1b59194520aba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# annotation-confidence-level-grading

## Summary

Assigns Schymanski et al. annotation confidence levels to untargeted metabolomics features using GetFeatistics, with MoNAScore cut-offs for patRoon imports and dot-product plus fragment-presence cut-offs for MS-DIAL imports.

## When to use

Use when you have a pre-processed untargeted feature table from patRoon or MS-DIAL and need one Schymanski confidence level per annotated feature before statistics or reporting.

## When NOT to use

- The study is targeted and quantification is calibration-curve based — GetFeatistics handles that via a separate targeted workflow, not annotation-level grading
- The feature table was produced by a tool other than patRoon or MS-DIAL (e.g. raw XCMS output), since the dedicated get_AnnoLevels_MSDial / get_feat_info_from_patRoon functions expect those two formats
- Features have no annotations at all — confidence-level grading operates on existing annotations, not on blank/QC filtering

## Inputs

- Pre-processed feature table exported from MS-DIAL (annotations with MS/MS dot-product and fragment information)
- Pre-processed feature table / annotation output exported from patRoon (annotations with MoNAScore)

## Outputs

- annoLevels.csv — one Schymanski et al. annotation confidence level per feature
- In-memory R feature table with an added annotation-confidence-level column

## How to apply

Import the pre-processed feature table into the GetFeatistics R environment (package installable via devtools::install_github from FrigerioGianfranco/GetFeatistics, R ≥ 4.3.1). For MS-DIAL tables, call get_AnnoLevels_MSDial, which grades each annotation with dot-product similarity cut-offs combined with fragment-presence checks and maps results onto Schymanski et al. confidence levels. For patRoon tables, call get_feat_info_from_patRoon, which grades annotations using MoNAScore cut-offs mapped to the same Schymanski levels. Write the graded annotations, one confidence level per feature, to annoLevels.csv. The rationale is that patRoon and MS-DIAL encode annotation quality differently (MoNAScore vs. dot product + fragment presence), so GetFeatistics provides dedicated import functions that normalise both onto a common confidence scale.

## Related tools

- **GetFeatistics** (R package whose get_AnnoLevels_MSDial and get_feat_info_from_patRoon functions grade annotations onto Schymanski confidence levels and write annoLevels.csv) — https://github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Source of imported feature tables whose annotations are graded with MoNAScore cut-offs; patRoon itself also offers automatic estimation of confidence levels natively) — https://github.com/rickhelmus/patRoon
- **MS-DIAL** (Source of imported feature tables whose annotations are graded with dot-product similarity and fragment-presence cut-offs)
- **R** (Runtime environment for GetFeatistics (version ≥ 4.3.1 required, with matching-architecture Java))

## Examples

```
library(GetFeatistics); annoLevels <- get_AnnoLevels_MSDial(msdial_feature_table); write.csv(annoLevels, "annoLevels.csv")
```

## Evaluation signals

- annoLevels.csv exists and contains exactly one confidence level per feature (no features left ungraded)
- Confidence levels conform to the Schymanski et al. level nomenclature
- patRoon-derived grades are reproducible from MoNAScore cut-offs and MS-DIAL grades from dot-product + fragment-presence cut-offs (spot-check a few features against source annotation scores)
- Feature count in annoLevels.csv matches the row count of the imported feature table (no features dropped during grading)

## Limitations

- Cut-off grading is only implemented for the two supported import paths (patRoon MoNAScore; MS-DIAL dot product + fragment presence); other annotation sources are not covered
- The grading is cut-off-based automation, so borderline annotations may land at a lower confidence level than manual expert curation would assign

## Evidence

- [abstract] Dedicated functions for both importers: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [readme] patRoon native confidence estimation: [evidence span withheld — anchor status: external_doc]
- [readme] GetFeatistics installation route: "devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE)"
