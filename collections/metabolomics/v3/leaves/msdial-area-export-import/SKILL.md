---
name: msdial-area-export-import
description: Use when untargeted metabolomics feature data has been processed in MS-DIAL
  and needs to be brought into GetFeatistics as featTable/featINFO for QC filtering,
  annotation confidence assignment, and statistics.
when_to_use_negative:
- The input feature table was produced by a tool other than MS-DIAL or patRoon (e.g.
  raw data not yet processed by XCMS/MS-DIAL — extract features first)
- The data are targeted-study intensities destined for calibration curve-based quantification
  rather than untargeted feature-table import
- You need generic MS-DIAL post-processing outside the GetFeatistics featTable/featINFO
  schema — the functions are package-specific
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GetFeatistics
  role: Provides get_feat_table_from_MSDial and get_feat_info_from_MSDial, which structure
    the MS-DIAL area export into featTable and featINFO; also provides the mock examples
    used to validate the import
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: MS-DIAL
  role: Upstream untargeted feature-detection tool whose exported feature table is
    the direct input to the import functions
- name: R
  role: Runtime environment in which GetFeatistics is installed (>= 4.3.1, with same-architecture
    Java) and the import functions are executed
- name: patRoon
  role: 'Alternative upstream source: get_feat_table_from_patRoon and get_feat_info_from_patRoon
    import featureGroups and MFsummary objects into the same featTable/featINFO structures'
  repo: https://github.com/rickhelmus/patRoon
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:ef4f4671e55e32db03dd085fc487b278777b382f1b7e726ac15bf43441e13f5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
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
yes_rate: 0.8
n_claims: 10
---

# msdial-area-export-import

## Summary

Imports an MS-DIAL area export into GetFeatistics by structuring feature intensities and metadata into the package's two core data frames, featTable and featINFO, via get_feat_table_from_MSDial and get_feat_info_from_MSDial.

## When to use

Use when untargeted metabolomics feature data has been processed in MS-DIAL and needs to be brought into GetFeatistics as featTable/featINFO for QC filtering, annotation confidence assignment, and statistics.

## When NOT to use

- The input feature table was produced by a tool other than MS-DIAL or patRoon (e.g. raw data not yet processed by XCMS/MS-DIAL — extract features first)
- The data are targeted-study intensities destined for calibration curve-based quantification rather than untargeted feature-table import
- You need generic MS-DIAL post-processing outside the GetFeatistics featTable/featINFO schema — the functions are package-specific

## Inputs

- MS-DIAL area export file (feature intensities across samples)
- GetFeatistics package installed in R (>= 4.3.1) with devtools and Java prerequisites
- df_example_feat_intensities and df_example_feat_info mock structures shipped with the package (for validation)

## Outputs

- featTable data frame (feature intensities, samples aligned to features)
- featINFO data frame (feature metadata)
- structures ready for GetFeatistics pooled-QC filtering and downstream statistics

## How to apply

Load the GetFeatistics package in R (>= 4.3.1, with matching-architecture Java) after installing it with devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE). Load the MS-DIAL area export containing feature intensities across samples and run get_feat_table_from_MSDial and get_feat_info_from_MSDial to build the featTable (feature intensities matrix) and featINFO (feature metadata) data frames. Verify the imported structures against the package's mock examples df_example_feat_intensities and df_example_feat_info, checking field names, value types, and feature/sample alignment. Once featTable and featINFO are correctly formed, downstream GetFeatistics steps (pooled-QC filtering, annotation confidence levels, regression modelling) can proceed. The same pattern applies to patRoon via get_feat_table_from_patRoon and get_feat_info_from_patRoon, taking featureGroups and MFsummary objects as input.

## Related tools

- **GetFeatistics** (Provides get_feat_table_from_MSDial and get_feat_info_from_MSDial, which structure the MS-DIAL area export into featTable and featINFO; also provides the mock examples used to validate the import) — https://github.com/FrigerioGianfranco/GetFeatistics
- **MS-DIAL** (Upstream untargeted feature-detection tool whose exported feature table is the direct input to the import functions)
- **R** (Runtime environment in which GetFeatistics is installed (>= 4.3.1, with same-architecture Java) and the import functions are executed)
- **patRoon** (Alternative upstream source: get_feat_table_from_patRoon and get_feat_info_from_patRoon import featureGroups and MFsummary objects into the same featTable/featINFO structures) — https://github.com/rickhelmus/patRoon

## Examples

```
library(GetFeatistics); featTable <- get_feat_table_from_MSDial("msdial_area_export.csv"); featINFO <- get_feat_info_from_MSDial("msdial_area_export.csv")
```

## Evaluation signals

- Imported featTable matches df_example_feat_intensities: field names and value types present as in the mock example
- Imported featINFO matches df_example_feat_info: metadata fields, names, and types align with the package mock
- Feature and sample alignment is preserved between featTable and featINFO (features in the MS-DIAL export map one-to-one)
- Only the two core data frames featTable and featINFO are produced, matching the package's documented import mechanism

## Limitations

- Import is schema-specific: outputs must conform exactly to the df_example_feat_intensities/df_example_feat_info structures, so malformed or non-standard MS-DIAL exports will fail alignment checks
- Requires R >= 4.3.1 and Java installed with the same architecture (64-bit or 32-bit) as R before installation
- For patRoon input, specific objects (featureGroups, MFsummary) must be available; the import functions do not operate on arbitrary tables

## Evidence

- [abstract] For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL"
- [results] The package provides dedicated import functions for patRoon and MS-DIAL that structure feature intensities and metadata into featTable and featINFO: [evidence span withheld — anchor status: not_found]
- [results] The imported data frames must match the df_example_feat_intensities and df_example_feat_info structures: [evidence span withheld — anchor status: not_found]
- [readme] GetFeatistics README on installation prerequisites and usage: [evidence span withheld — anchor status: not_found]
