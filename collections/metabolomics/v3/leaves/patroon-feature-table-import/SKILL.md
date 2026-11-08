---
name: patroon-feature-table-import
description: Use when patRoon featureGroups and MFsummary outputs must be brought
  into GetFeatistics as featTable/featINFO data frames for pooled-QC filtering, annotation
  grading, regression modelling, or plotting.
when_to_use_negative:
- The input is an MS-DIAL area export rather than patRoon output — use get_feat_table_from_MSDial
  / get_feat_info_from_MSDial, which apply dot-product and fragment-presence cut-offs
  instead of MoNAScore
- Raw MS data still needs feature extraction — patRoon (or XCMS) must first process
  the raw data files before tables exist to import
- A targeted study requiring calibration curve-based quantification with weighted
  regressions is the goal rather than untargeted feature-table elaboration
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: GetFeatistics
  role: Provides get_feat_table_from_patRoon and get_feat_info_from_patRoon import
    functions that build featTable and featINFO and grade annotations to Schymanski
    levels via MoNAScore cut-offs
  repo: github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Upstream non-target analysis tool producing the featureGroups and MFsummary
    objects consumed by the import functions
  repo: rickhelmus/patRoon
- name: R
  role: Runtime environment (version >= 4.3.1) in which GetFeatistics and the import
    functions are executed
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:74f8e6dff3ce05c869ffba04a4cfd80e0d035007cfc747ca51db1caf79c6fd5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# patroon-feature-table-import

## Summary

Import patRoon feature-group results (featureGroups and MFsummary objects) into the GetFeatistics R environment using get_feat_table_from_patRoon and get_feat_info_from_patRoon, structuring intensities and metadata into the featTable and featINFO data frames that all downstream GetFeatistics processing expects.

## When to use

Use when patRoon featureGroups and MFsummary outputs must be brought into GetFeatistics as featTable/featINFO data frames for pooled-QC filtering, annotation grading, regression modelling, or plotting.

## When NOT to use

- The input is an MS-DIAL area export rather than patRoon output — use get_feat_table_from_MSDial / get_feat_info_from_MSDial, which apply dot-product and fragment-presence cut-offs instead of MoNAScore
- Raw MS data still needs feature extraction — patRoon (or XCMS) must first process the raw data files before tables exist to import
- A targeted study requiring calibration curve-based quantification with weighted regressions is the goal rather than untargeted feature-table elaboration

## Inputs

- patRoon featureGroups object (feature grouping across analyses)
- patRoon MFsummary object (formula/annotation summary)
- R environment (version >= 4.3.1) with GetFeatistics installed and Java with the same architecture as R

## Outputs

- featTable data frame of feature intensities per sample
- featINFO data frame of per-feature metadata and annotations
- annoLevels.csv with one Schymanski et al. confidence level per feature (MoNAScore cut-offs)

## How to apply

Install GetFeatistics in R (version >= 4.3.1, with Java matching R's architecture) via devtools::install_github('FrigerioGianfranco/GetFeatistics', dependencies = TRUE), then library(GetFeatistics). Construct the patRoon featureGroups and MFsummary objects and call get_feat_table_from_patRoon and get_feat_info_from_patRoon so feature intensities and metadata are structured into the two core data frames featTable (sample x feature intensities) and featINFO (per-feature metadata). While importing, the get_feat_info_from_patRoon function grades annotations using MoNAScore cut-offs mapped onto Schymanski et al. confidence levels, writing one confidence level per feature to annoLevels.csv. Validate the imported data frames against the package's df_example_feat_intensities and df_example_feat_info mock structures to confirm column names, value types, and feature/sample alignment before proceeding to QC filtration or statistics.

## Related tools

- **GetFeatistics** (Provides get_feat_table_from_patRoon and get_feat_info_from_patRoon import functions that build featTable and featINFO and grade annotations to Schymanski levels via MoNAScore cut-offs) — github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Upstream non-target analysis tool producing the featureGroups and MFsummary objects consumed by the import functions) — rickhelmus/patRoon
- **R** (Runtime environment (version >= 4.3.1) in which GetFeatistics and the import functions are executed)

## Examples

```
library(GetFeatistics); featTable <- get_feat_table_from_patRoon(featureGroups); featINFO <- get_feat_info_from_patRoon(MFsummary)
```

## Evaluation signals

- Imported featTable and featINFO match the df_example_feat_intensities and df_example_feat_info mock structures in field names and value types
- Feature and sample alignment is preserved: every feature in featTable has a corresponding row in featINFO and vice versa
- annoLevels.csv contains exactly one confidence level per feature, with grades consistent with MoNAScore cut-offs mapped to Schymanski et al. levels
- The package installs fully with devtools::install_github('FrigerioGianfranco/GetFeatistics', dependencies = TRUE) and library(GetFeatistics) loads without error

## Limitations

- Import requires pre-processed patRoon objects — GetFeatistics does not run the patRoon workflow itself
- Java must be installed with the same architecture (64-bit or 32-bit) as R, or installation/use will fail
- The annotation grading for patRoon imports relies on MoNAScore cut-offs only (dot-product and fragment-presence checks apply to MS-DIAL imports, not patRoon)

## Evidence

- [abstract] For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL, assign annotation confidence levels"
- [intro] The GetFeatistics package has been developed with the R-language and is publicly available on GitHub: "The GetFeatistics package has been developed with the R-language and is publicly available on GitHub"
- [readme] Before installing, ensure the following are installed: R (version >= 4.3.1), Java, with the same architecture as R (64-bit or 32-bit).: "Before installing, ensure the following are installed:
- R (version ≥ 4.3.1)
- Java, with the same architecture as R (64-bit or 32-bit)."
