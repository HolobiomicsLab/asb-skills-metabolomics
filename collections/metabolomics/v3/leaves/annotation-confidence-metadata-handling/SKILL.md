---
name: annotation-confidence-metadata-handling
description: Use when untargeted metabolomics feature tables produced by patRoon (featureGroups/MFsummary
  objects) or MS-DIAL (area exports) need to be imported into R with annotation confidence
  levels assigned before QC filtering and statistics.
when_to_use_negative:
- The feature table is already a structured featTable/featINFO data frame — re-importing
  risks misaligning samples and features
- 'The study is targeted: calibration curve-based quantification with weighting options
  is used instead of untargeted import functions'
- Raw mass spectrometry data still needs feature extraction — use patRoon/XCMS processing
  first; these functions only import existing feature tables
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3174
tools:
- name: GetFeatistics
  role: Provides the import functions (get_feat_table_from_patRoon, get_feat_info_from_patRoon,
    get_feat_table_from_MSDial, get_feat_info_from_MSDial) that assign annotation
    confidence levels and build featTable/featINFO
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Upstream non-target analysis tool producing featureGroups and MFsummary objects
    that GetFeatistics imports; also natively estimates identification confidence
    levels
  repo: rickhelmus/patRoon
- name: MS-DIAL
  role: Upstream feature detection tool whose area exports are parsed by the GetFeatistics
    MS-DIAL import functions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:042bef542643a63d3b28ff374a94d00c46f1856528bc662d786e7d4ed6309825
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# annotation-confidence-metadata-handling

## Summary

Assigns annotation confidence levels to untargeted metabolomics features and structures feature intensities and metadata from patRoon or MS-DIAL outputs into GetFeatistics' two core data frames (featTable and featINFO), so that downstream QC filtering and statistics operate on consistently labelled features.

## When to use

Use when untargeted metabolomics feature tables produced by patRoon (featureGroups/MFsummary objects) or MS-DIAL (area exports) need to be imported into R with annotation confidence levels assigned before QC filtering and statistics.

## When NOT to use

- The feature table is already a structured featTable/featINFO data frame — re-importing risks misaligning samples and features
- The study is targeted: calibration curve-based quantification with weighting options is used instead of untargeted import functions
- Raw mass spectrometry data still needs feature extraction — use patRoon/XCMS processing first; these functions only import existing feature tables

## Inputs

- patRoon featureGroups object (R)
- patRoon MFsummary object (R)
- MS-DIAL feature area export table
- GetFeatistics installed in R >= 4.3.1 with Java matching R architecture

## Outputs

- featTable data frame of feature intensities (samples × features)
- featINFO data frame of feature metadata including annotation confidence levels

## How to apply

Load GetFeatistics in R (≥ 4.3.1) and run the dedicated import functions: get_feat_table_from_patRoon and get_feat_info_from_patRoon on patRoon featureGroups and MFsummary objects, or get_feat_table_from_MSDial and get_feat_info_from_MSDial on an MS-DIAL area export. These functions split the source output into two data frames: featTable holding feature intensities per sample and featINFO holding feature metadata (including annotation confidence levels assigned by the package). Decision point: the source tool determines which import pair to call — patRoon objects must exist in the R session, while MS-DIAL input is a spreadsheet export. Validate the resulting structures against the package's df_example_feat_intensities and df_example_feat_info mocks, checking column names, value types, and feature/sample alignment, before proceeding to pooled QC filtering.

## Related tools

- **GetFeatistics** (Provides the import functions (get_feat_table_from_patRoon, get_feat_info_from_patRoon, get_feat_table_from_MSDial, get_feat_info_from_MSDial) that assign annotation confidence levels and build featTable/featINFO) — github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Upstream non-target analysis tool producing featureGroups and MFsummary objects that GetFeatistics imports; also natively estimates identification confidence levels) — rickhelmus/patRoon
- **MS-DIAL** (Upstream feature detection tool whose area exports are parsed by the GetFeatistics MS-DIAL import functions)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Imported featTable matches the df_example_feat_intensities mock structure (field names and types present as in the mock example)
- Imported featINFO matches the df_example_feat_info mock structure (field names and types present as in the mock example)
- Feature and sample alignment preserved: rows/columns of featTable correspond 1:1 to features listed in featINFO
- Every feature carries an assigned annotation confidence level after import
- Metadata types check: intensity columns are numeric, identifier/metadata columns conform to the example schema

## Limitations

- Confidence levels are assigned within the untargeted workflow only; patRoon's confidence estimation depends on upstream annotation tool results (e.g. MetFrag/SIRIUS compound annotation)
- Import correctness depends on matching GetFeatistics' expected schemas; divergent MS-DIAL export formats or missing MFsummary fields will break column alignment
- featTable/featINFO must be built before pooled QC filtering can be applied, so import errors propagate to the entire downstream pipeline

## Evidence

- [abstract] Dedicated import functions assign confidence levels and import patRoon/MS-DIAL feature tables: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels, and filter features based on pooled quality"
- [readme] patRoon natively estimates identification confidence levels: [evidence span withheld — anchor status: external_doc]
- [readme] GetFeatistics is installed from GitHub and loaded in R: "devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE)"
