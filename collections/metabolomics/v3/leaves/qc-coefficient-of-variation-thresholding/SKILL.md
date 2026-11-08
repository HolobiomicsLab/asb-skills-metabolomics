---
name: qc-coefficient-of-variation-thresholding
description: Use when an untargeted metabolomics feature table (e.g. exported from
  patRoon or MS-DIAL) contains blank, pooled QC, and diluted-QC (QC_half) sample groups
  and low-quality features must be removed before statistical analysis.
when_to_use_negative:
- The study is targeted with calibration-curve-based quantification, which GetFeatistics
  handles through a different quantification pathway rather than pooled-QC feature
  filtering.
- The feature table does not contain blank, pooled QC, or diluted-QC (QC_half) sample
  groups, so the four QC criteria cannot be computed.
- Features have already been filtered by pooled QC criteria upstream and only downstream
  steps (intensity pre-processing, regression modelling) remain.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: GetFeatistics
  role: R-package whose QCs_process function implements the four-step pooled-QC feature
    filtering and the separated per-sample-group pooled-QC merge
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime environment (R >= 4.3.1) in which GetFeatistics is installed and the
    feature table is loaded and filtered
- name: patRoon
  role: Upstream tool whose exported feature tables can be imported into GetFeatistics
    as the filtering input
  repo: rickhelmus/patRoon
- name: MS-DIAL
  role: Alternative upstream feature-detection tool whose feature tables can be imported
    into GetFeatistics for QC filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:3d5ba204d7bdb56d663ea4c04ab0104941c50ed2e738813b454c5d2e1574140d
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
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# qc-coefficient-of-variation-thresholding

## Summary

Filter untargeted metabolomics features using pooled quality control criteria as implemented in the GetFeatistics R-package's QCs_process function: QC reproducibility (CV%), QC detection frequency, blank contribution, and diluted-QC response, with an optional per-sample-group separated pooled-QC variant.

## When to use

Use when an untargeted metabolomics feature table (e.g. exported from patRoon or MS-DIAL) contains blank, pooled QC, and diluted-QC (QC_half) sample groups and low-quality features must be removed before statistical analysis.

## When NOT to use

- The study is targeted with calibration-curve-based quantification, which GetFeatistics handles through a different quantification pathway rather than pooled-QC feature filtering.
- The feature table does not contain blank, pooled QC, or diluted-QC (QC_half) sample groups, so the four QC criteria cannot be computed.
- Features have already been filtered by pooled QC criteria upstream and only downstream steps (intensity pre-processing, regression modelling) remain.

## Inputs

- Feature table from untargeted metabolomics processing (importable from patRoon or MS-DIAL) containing blank, pooled QC, and diluted-QC (QC_half) sample groups
- Sample-group metadata labeling which injections are blanks, pooled QCs, and QC_half diluted QCs
- QC cut-off thresholds (CV% cut-off, minimum QC detection frequency, acceptable blank/QC ratio, acceptable QC/QC_half dilution-response range)

## Outputs

- Per-step pass/fail results for each feature across the four QC criteria (written to a CSV file)
- Final retained-feature table containing only features that pass the pooled QC filtering (written to a CSV file)
- For the separated pooled-QC variant, a merged feature set of features passing in at least one QC group

## How to apply

Load the constructed feature table with blank, QC, and diluted-QC (QC_half) sample groups into R and run the GetFeatistics QCs_process filtering. Step I: compute the relative standard deviation (CV%) of each feature's intensity across pooled QC injections and flag features exceeding the CV cut-off (poor reproducibility). Step II: compute each feature's detection frequency across QC injections and flag features with insufficient QC presence. Step III: compute the blank/QC mean intensity ratio and flag features whose QC signal is not sufficiently higher than blank contribution. Step IIII: compute the QC/QC_half mean intensity ratio and flag features whose response does not follow the expected fold-change upon dilution. For studies with separated pooled QCs per sample group, compute the QC criteria per group and retain features that survive the criteria in at least one QC group (the merge step). Rationale: pooled QCs monitor analytical reproducibility throughout acquisition, so only features passing all four criteria are trustworthy for downstream statistics.

## Related tools

- **GetFeatistics** (R-package whose QCs_process function implements the four-step pooled-QC feature filtering and the separated per-sample-group pooled-QC merge) — github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime environment (R >= 4.3.1) in which GetFeatistics is installed and the feature table is loaded and filtered)
- **patRoon** (Upstream tool whose exported feature tables can be imported into GetFeatistics as the filtering input) — rickhelmus/patRoon
- **MS-DIAL** (Alternative upstream feature-detection tool whose feature tables can be imported into GetFeatistics for QC filtering)

## Evaluation signals

- The per-step CSV output exists and records, for each feature, a pass/fail flag for each of the four criteria: QC CV%, QC detection frequency, blank/QC ratio, and QC/QC_half dilution-response ratio.
- Every feature in the final retained-feature CSV passes all four QC criteria (or, for the sep_QC variant, passes the criteria in at least one separated QC group).
- Features removed in step I have QC-intensity CV% above the chosen cut-off, i.e. their relative standard deviation across pooled QC injections exceeds the reproducibility threshold.
- Features removed in step III have a blank/QC mean intensity ratio within the unacceptable range, i.e. QC signal not sufficiently above blank.
- Features removed in step IIII have a QC/QC_half intensity ratio outside the acceptable range for the expected dilution fold-change.

## Limitations

- The filtering requires pooled QC, blank, and diluted-QC (QC_half) samples to have been included during sample preparation; without these injections the criteria cannot be evaluated.
- Cut-off values for CV%, detection frequency, blank/QC ratio, and dilution-response acceptability must be chosen by the user; the package flags features against the supplied thresholds rather than prescribing them.
- With separated pooled QCs per sample group, the merge keeps features passing in at least one QC group, which can retain features that fail in other groups.

## Evidence

- [abstract] For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [intro] Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed: "Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed"
- [discussion] the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group: "the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group"
- [intro] the inclusion of pooled quality control (QC) samples during sample preparation is a key step to monitor data quality and analytical reproducibility: "the inclusion of pooled quality control (QC) samples during sample preparation is a key step to monitor data quality and analytical reproducibility"
- [readme] Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables, separate QC processing"
