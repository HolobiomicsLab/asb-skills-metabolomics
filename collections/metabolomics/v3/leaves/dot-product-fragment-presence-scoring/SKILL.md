---
name: dot-product-fragment-presence-scoring
description: Use when a patRoon or MS-DIAL feature table has been imported into GetFeatistics
  and each annotated feature needs an automatic Schymanski-style annotation confidence
  level derived from MS/MS match quality (MoNAScore for patRoon; dot product plus
  fragment presence for MS-DIAL).
when_to_use_negative:
- The input is raw data still awaiting feature extraction (e.g. XCMS) rather than
  an already annotated patRoon/MS-DIAL feature table — annotation levels cannot be
  computed without existing annotations.
- A targeted-analysis workflow with calibration-curve quantification, which does not
  rely on untargeted annotation confidence grading.
- Feature tables from tools other than patRoon or MS-DIAL, for which GetFeatistics
  provides no dedicated annotation-level function.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GetFeatistics
  role: 'R package that grades annotations: get_AnnoLevels_MSDial (dot-product + fragment-presence
    cut-offs) and get_feat_info_from_patRoon (MoNAScore cut-offs), mapped to Schymanski
    confidence levels'
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Source of non-target feature tables and compound annotations whose MoNAScore-based
    confidence levels GetFeatistics re-grades
  repo: rickhelmus/patRoon
- name: MS-DIAL
  role: Source of feature tables whose annotations are graded via dot-product similarity
    cut-offs combined with fragment-presence checks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:ff5b4c1034ab2485b0dba764935f90955d8c556dee832a7f413650342116d560
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dot-product-fragment-presence-scoring

## Summary

GetFeatistics automatically assigns annotation confidence levels (Schymanski et al. scale) to features imported from MS-DIAL using dot-product similarity cut-offs combined with fragment-presence checks, and to patRoon features using MoNAScore cut-offs, writing one confidence level per feature to annoLevels.csv.

## When to use

Use when a patRoon or MS-DIAL feature table has been imported into GetFeatistics and each annotated feature needs an automatic Schymanski-style annotation confidence level derived from MS/MS match quality (MoNAScore for patRoon; dot product plus fragment presence for MS-DIAL).

## When NOT to use

- The input is raw data still awaiting feature extraction (e.g. XCMS) rather than an already annotated patRoon/MS-DIAL feature table — annotation levels cannot be computed without existing annotations.
- A targeted-analysis workflow with calibration-curve quantification, which does not rely on untargeted annotation confidence grading.
- Feature tables from tools other than patRoon or MS-DIAL, for which GetFeatistics provides no dedicated annotation-level function.

## Inputs

- Pre-processed MS-DIAL feature/annotation table (MS/MS dot-product and fragment information present)
- Pre-processed patRoon feature table with annotation results (compound annotations with MoNAScore values)

## Outputs

- annoLevels.csv — one Schymanski et al. annotation confidence level per feature

## How to apply

First import the pre-processed feature table into the GetFeatistics R environment; untargeted feature tables come from patRoon or MS-DIAL. For MS-DIAL tables, call get_AnnoLevels_MSDial so each annotation is graded by dot-product similarity cut-offs combined with fragment-presence checks, and the result is mapped onto the Schymanski et al. confidence levels. For patRoon tables, call get_feat_info_from_patRoon so annotations are graded with MoNAScore cut-offs mapped to the same Schymanski confidence levels. Finally, write the graded annotations to annoLevels.csv with exactly one confidence level per feature. The rationale is that metabolite annotation in untargeted workflows (e.g. via MetFrag or PubChemLite) is heterogeneous in quality, so grading each feature by its MS/MS evidence makes downstream interpretation and reporting consistent.

## Related tools

- **GetFeatistics** (R package that grades annotations: get_AnnoLevels_MSDial (dot-product + fragment-presence cut-offs) and get_feat_info_from_patRoon (MoNAScore cut-offs), mapped to Schymanski confidence levels) — github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Source of non-target feature tables and compound annotations whose MoNAScore-based confidence levels GetFeatistics re-grades) — rickhelmus/patRoon
- **MS-DIAL** (Source of feature tables whose annotations are graded via dot-product similarity cut-offs combined with fragment-presence checks)

## Examples

```
library(GetFeatistics); annoLevels <- get_AnnoLevels_MSDial(msdial_table); write.csv(annoLevels, "annoLevels.csv")
```

## Evaluation signals

- annoLevels.csv exists and contains exactly one confidence level per feature in the imported table (no ungraded annotated features).
- For MS-DIAL inputs, spot-check that assigned Schymanski levels are consistent with the dot-product similarity cut-offs and fragment-presence checks implemented by get_AnnoLevels_MSDial.
- For patRoon inputs, verify that graded levels follow the MoNAScore cut-off mapping to Schymanski levels applied by get_feat_info_from_patRoon.
- Every output level is a valid Schymanski et al. annotation confidence level (no arbitrary or numeric-only grades).
- Re-running the grading on the same feature table yields an identical annoLevels.csv (deterministic cut-off mapping).

## Limitations

- Only patRoon and MS-DIAL feature tables are supported by the dedicated import/grading functions; other annotation sources are not covered.
- Grading is fully automatic and threshold-based (MoNAScore / dot product / fragment presence), so it cannot incorporate manual expert curation of individual annotations.
- Features without annotations cannot be graded — the confidence level reflects MS/MS annotation evidence only.

## Evidence

- [abstract] dedicated functions to import feature tables and grade annotations: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [intro] annotation context in untargeted workflows: "annotation software such as MetFrag [6] and databases like PubChemLite [7] support metabolite annotation"
- [readme] function-level documentation availability: [evidence span withheld — anchor status: external_doc]
- [other] patRoon-side confidence estimation: [evidence span withheld — anchor status: external_doc]
