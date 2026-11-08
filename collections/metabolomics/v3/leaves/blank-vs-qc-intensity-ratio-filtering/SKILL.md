---
name: blank-vs-qc-intensity-ratio-filtering
description: Use when a feature table from an untargeted LC-MS metabolomics run contains
  both procedural blank and pooled QC sample groups and features must be filtered
  on QC quality cut-off criteria before statistics.
when_to_use_negative:
- The study is targeted with calibration-curve-based quantification, where GetFeatistics
  applies quantification with weighting options instead of pooled-QC filtration
- The acquisition contains no procedural blank injections, so a blank/QC intensity
  ratio cannot be computed
- The feature table has already been filtered on pooled QC criteria and no re-filtering
  of blank contribution is intended
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3339
tools:
- name: GetFeatistics
  role: R-package providing the QCs_process function that computes the blank/QC mean
    intensity ratio cut-off as one of the pooled-QC filtration criteria
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: upstream tool whose exported feature tables can be imported into GetFeatistics
    for QC-based filtration
  repo: rickhelmus/patRoon
- name: MS-DIAL
  role: alternative upstream feature-detection tool whose feature tables can be imported
    into GetFeatistics for QC-based filtration
- name: R
  role: runtime environment (version >= 4.3.1) in which the filtration is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:ef81639e15b906356b3c7a156fa052fea41d8e405bd5144e2fd3d4f67b8fd0d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# blank-vs-qc-intensity-ratio-filtering

## Summary

A pooled-QC feature filtering criterion from GetFeatistics' QCs_process that flags untargeted metabolomics features whose mean intensity in blanks is too high relative to pooled QCs (high blank/QC mean intensity ratio), removing features attributable to background contamination rather than biology.

## When to use

Use when a feature table from an untargeted LC-MS metabolomics run contains both procedural blank and pooled QC sample groups and features must be filtered on QC quality cut-off criteria before statistics.

## When NOT to use

- The study is targeted with calibration-curve-based quantification, where GetFeatistics applies quantification with weighting options instead of pooled-QC filtration
- The acquisition contains no procedural blank injections, so a blank/QC intensity ratio cannot be computed
- The feature table has already been filtered on pooled QC criteria and no re-filtering of blank contribution is intended

## Inputs

- Feature table with grouped intensities including blank, pooled QC, and diluted-QC (QC_half) sample groups (e.g. imported from patRoon or MS-DIAL)
- User-defined blank/QC mean intensity ratio cut-off
- Sample group annotations distinguishing blanks, QCs, and QC_half injections

## Outputs

- Filtered feature table retaining only features whose blank/QC mean intensity ratio passes the cut-off
- Per-step pass/fail table indicating which features failed the blank-contribution criterion vs the other QC criteria (CSV)

## How to apply

Within the GetFeatistics QCs_process four-step filtration, this is the third criterion: for every feature, compute the ratio of its mean intensity in blank samples to its mean intensity in pooled QC samples, and flag features whose intensity is not sufficiently higher in QCs than in blanks. This complements the other pooled-QC criteria applied in the same pass — the CV% (relative standard deviation) of QC intensities (reproducibility), the detection frequency of the feature across QC injections, and the QC vs diluted-QC (QC_half) mean intensity ratio (dilution behaviour) — each with its own user-defined cut-off. Features failing this blank/QC criterion are removed from the feature table, alongside the other flagged features, before downstream intensity pre-processing and statistics. Per-step pass/fail results should be reported so users can trace which features were dropped at the blank-contribution step.

## Related tools

- **GetFeatistics** (R-package providing the QCs_process function that computes the blank/QC mean intensity ratio cut-off as one of the pooled-QC filtration criteria) — github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (upstream tool whose exported feature tables can be imported into GetFeatistics for QC-based filtration) — rickhelmus/patRoon
- **MS-DIAL** (alternative upstream feature-detection tool whose feature tables can be imported into GetFeatistics for QC-based filtration)
- **R** (runtime environment (version >= 4.3.1) in which the filtration is executed)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Every retained feature has a blank/QC mean intensity ratio below the configured cut-off, while flagged features are documented in the per-step pass/fail output
- The per-step pass/fail results and the final retained-feature table are written to CSV as specified by the QCs_process workflow
- Features removed at the blank-contribution step show intensities not sufficiently higher in QCs than in blanks, consistent with background contamination
- The filtration composes correctly with the other QCs_process criteria (QC CV%, QC detection frequency, QC/QC_half ratio) and the separated pooled-QC per-group merge, where a feature is kept if it survives the criteria in at least one QC group

## Limitations

- The criterion depends on the presence and quality of pooled QC and blank samples included during sample preparation; without them filtration cannot be performed
- Cut-offs are user-defined, so overly strict blank/QC thresholds may remove genuine low-abundance features
- When separated pooled QCs per sample group are used, a feature only needs to survive the criteria in at least one QC group at the merge step, which relaxes the per-group blank-contribution requirement

## Evidence

- [intro] Filtering based on QC quality cut-off criteria: "Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed"
- [discussion] Pooled QC filtration including separated pooled QC per sample group: "the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group"
- [abstract] GetFeatistics supports filtering features based on pooled quality: "and filter features based on pooled quality"
- [readme] GetFeatistics covers separate QC processing of metabolomics feature tables: "including elaboration of feature tables, separate QC processing"
