---
name: monascore-cutoff-classification
description: Use when a pre-processed feature table from patRoon or MS-DIAL needs
  annotation confidence levels (Schymanski scheme) assigned per feature, or when annotations
  must be graded before downstream QC filtering/statistics.
when_to_use_negative:
- The analysis is a targeted quantification workflow with calibration curves — confidence-level
  grading of untargeted annotations is not applicable.
- The feature table comes from an unsupported tool (e.g. raw XCMS output rather than
  patRoon or MS-DIAL), since the cut-off logic is specific to MoNAScore (patRoon)
  and dot-product/fragment-presence (MS-DIAL).
- Annotations have no associated MS/MS or fragment information, making dot-product
  and fragment-presence grading meaningless.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: GetFeatistics
  role: Provides get_AnnoLevels_MSDial and get_feat_info_from_patRoon functions that
    implement the cut-off logic and Schymanski level mapping, writing annoLevels.csv
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Source of pre-processed feature tables whose compound annotations are graded
    via MoNAScore cut-offs
  repo: https://github.com/rickhelmus/patRoon
- name: MS-DIAL
  role: Source of pre-processed feature tables whose annotations are graded via dot-product
    similarity and fragment-presence cut-offs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:f169fa557c80754055634320e7612238189342560dc8a48341739e2faa4d9e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# monascore-cutoff-classification

## Summary

Assigns Schymanski et al. annotation confidence levels to imported metabolomics annotations by applying MoNAScore cut-offs (patRoon tables) or dot-product plus fragment-presence cut-offs (MS-DIAL tables) using the GetFeatistics R-package.

## When to use

Use when a pre-processed feature table from patRoon or MS-DIAL needs annotation confidence levels (Schymanski scheme) assigned per feature, or when annotations must be graded before downstream QC filtering/statistics.

## When NOT to use

- The analysis is a targeted quantification workflow with calibration curves — confidence-level grading of untargeted annotations is not applicable.
- The feature table comes from an unsupported tool (e.g. raw XCMS output rather than patRoon or MS-DIAL), since the cut-off logic is specific to MoNAScore (patRoon) and dot-product/fragment-presence (MS-DIAL).
- Annotations have no associated MS/MS or fragment information, making dot-product and fragment-presence grading meaningless.

## Inputs

- Pre-processed MS-DIAL feature table (with MS/MS spectra and fragment annotations)
- Pre-processed patRoon feature table (with in silico compound annotations)

## Outputs

- annoLevels.csv: one annotation confidence level (Schymanski et al. scale) per feature

## How to apply

Import the pre-processed feature table (patRoon or MS-DIAL output) into the GetFeatistics R environment. For MS-DIAL tables, call get_AnnoLevels_MSDial, which grades each annotation against dot-product similarity cut-offs combined with fragment-presence checks, and maps the result onto the Schymanski et al. confidence levels. For patRoon tables, call get_feat_info_from_patRoon, which grades annotations using MoNAScore cut-offs mapped to the same Schymanski confidence levels. Write the graded annotations — one confidence level per feature — to annoLevels.csv. This grading runs after annotation (e.g. MetFrag/PubChemLite-style in silico annotation) and before pooled-QC feature filtering and statistical modelling in the GetFeatistics pipeline.

## Related tools

- **GetFeatistics** (Provides get_AnnoLevels_MSDial and get_feat_info_from_patRoon functions that implement the cut-off logic and Schymanski level mapping, writing annoLevels.csv) — https://github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Source of pre-processed feature tables whose compound annotations are graded via MoNAScore cut-offs) — https://github.com/rickhelmus/patRoon
- **MS-DIAL** (Source of pre-processed feature tables whose annotations are graded via dot-product similarity and fragment-presence cut-offs)

## Examples

```
library(GetFeatistics); anno <- get_AnnoLevels_MSDial(msdial_table); write.csv(anno, "annoLevels.csv")
```

## Evaluation signals

- annoLevels.csv exists and contains exactly one confidence level per feature in the input table (no features dropped or duplicated).
- Levels conform to the Schymanski et al. confidence scale, not an ad-hoc numeric score.
- patRoon-derived levels are reproducible from MoNAScore cut-offs; MS-DIAL-derived levels agree with the dot-product and fragment-presence cut-off logic.
- Feature identities in annoLevels.csv match the imported patRoon/MS-DIAL feature table row-for-row.
- Downstream GetFeatistics steps (pooled-QC filtering, statistics) can consume the graded table without schema errors.

## Limitations

- Cut-off logic is source-specific: only MoNAScore cut-offs are implemented for patRoon and only dot-product plus fragment-presence cut-offs for MS-DIAL, so other tools require adaptation.
- The confidence grading inherits the quality of the upstream annotations; features without sufficient MS/MS or fragment evidence cannot be graded beyond the lowest levels.
- Requires R (≥ 4.3.1) and the GetFeatistics package installed with all dependencies, per the README installation instructions.

## Evidence

- [other] GetFeatistics automatically assigns annotation confidence levels implementing MoNAScore cut-offs for patRoon, and dot product and fragment presence cut-offs for MS-DIAL.: [evidence span withheld — anchor status: not_found]
- [abstract] For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [intro] annotation software such as MetFrag and databases like PubChemLite support metabolite annotation: "annotation software such as MetFrag [6] and databases like PubChemLite [7] support metabolite annotation"
- [readme] GetFeatistics is publicly available on GitHub: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing"
