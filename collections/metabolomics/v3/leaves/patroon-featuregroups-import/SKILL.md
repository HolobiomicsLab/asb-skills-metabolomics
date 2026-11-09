---
name: patroon-featuregroups-import
description: Use when starting an untargeted metabolomics analysis whose features
  were extracted and annotated with patRoon, and the data must be converted into GetFeatistics'
  featTable/featINFO structures before pooled-QC filtration or regression modelling.
when_to_use_negative:
- The data originate from MS-DIAL rather than patRoon — use the MS-DIAL import functions
  (get_feat_table_from_MSDial, get_feat_info_from_MSDial) instead
- The feature table is already in the GetFeatistics featTable/featINFO format, e.g.
  from a targeted study with calibration-curve quantification
- Feature extraction itself still needs to be performed from raw data — patRoon (or
  XCMS/OpenMS) must first produce featureGroups before import
edam_operation: http://edamontology.org/operation_2409
edam_topics: []
tools:
- name: GetFeatistics
  role: Provides the import functions (get_feat_table_from_patRoon, get_feat_info_from_patRoon)
    that structure patRoon outputs into featTable and featINFO
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Upstream NTA workflow tool whose featureGroups and MFsummary objects are the
    import source
  repo: https://github.com/rickhelmus/patRoon
- name: R
  role: Runtime language for both patRoon and GetFeatistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:c47ff4754e2e6684396b7af8e6692c223b809b94a5188d9fa6e9b07512599e68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - MS-DIAL
  - R
  - patRoon
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# patroon-featuregroups-import

## Summary

Import patRoon non-target analysis outputs (featureGroups and MFsummary objects) into GetFeatistics' two core data frames, featTable (feature intensities per sample) and featINFO (feature metadata), so downstream GetFeatistics QC filtering and statistics can run on untargeted metabolomics data.

## When to use

Use when starting an untargeted metabolomics analysis whose features were extracted and annotated with patRoon, and the data must be converted into GetFeatistics' featTable/featINFO structures before pooled-QC filtration or regression modelling.

## When NOT to use

- The data originate from MS-DIAL rather than patRoon — use the MS-DIAL import functions (get_feat_table_from_MSDial, get_feat_info_from_MSDial) instead
- The feature table is already in the GetFeatistics featTable/featINFO format, e.g. from a targeted study with calibration-curve quantification
- Feature extraction itself still needs to be performed from raw data — patRoon (or XCMS/OpenMS) must first produce featureGroups before import

## Inputs

- patRoon featureGroups object (feature intensities grouped across analyses from an NTA workflow)
- patRoon MFsummary object (compound/formula annotation summary, optional but used for featINFO)
- R >= 4.3.1 environment with the GetFeatistics package installed

## Outputs

- featTable data frame of feature intensities (features x samples) matching the df_example_feat_intensities schema
- featINFO data frame of per-feature metadata matching the df_example_feat_info schema

## How to apply

Install GetFeatistics from GitHub (devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE)) and load it with library(GetFeatistics). From a completed patRoon workflow, take the featureGroups object (feature intensities grouped across analyses) and, if available, the MFsummary annotation object, and pass them to the dedicated import functions get_feat_table_from_patRoon and get_feat_info_from_patRoon. These restructure patRoon's intensities and metadata into featTable (rows = features, columns = samples, values = intensities) and featINFO (per-feature metadata such as annotations and confidence levels). Validate the result by comparing column names, value types, and feature/sample alignment against the package's mock structures df_example_feat_intensities and df_example_feat_info; once the tables match the expected schema, the data are ready for GetFeatistics' pooled-QC-based feature filtering, annotation confidence assignment, and statistical modelling.

## Related tools

- **GetFeatistics** (Provides the import functions (get_feat_table_from_patRoon, get_feat_info_from_patRoon) that structure patRoon outputs into featTable and featINFO) — https://github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Upstream NTA workflow tool whose featureGroups and MFsummary objects are the import source) — https://github.com/rickhelmus/patRoon
- **R** (Runtime language for both patRoon and GetFeatistics)

## Examples

```
library(GetFeatistics); featTable <- get_feat_table_from_patRoon(featureGroups); featINFO <- get_feat_info_from_patRoon(featureGroups, MFsummary)
```

## Evaluation signals

- Imported featTable matches df_example_feat_intensities in field names and value types
- Imported featINFO matches df_example_feat_info in structure and content type
- Feature and sample alignment is preserved: the same features/samples present in the patRoon featureGroups object appear in featTable with consistent identifiers
- Intensity values in featTable correspond numerically to the source patRoon featureGroups intensities (no scaling or reordering introduced by import)
- Downstream GetFeatistics steps (pooled-QC filtering, confidence-level assignment) accept the imported tables without schema errors

## Limitations

- Import requires patRoon objects already produced by a completed NTA workflow; GetFeatistics does not perform feature extraction itself
- MS-DIAL and patRoon each have their own dedicated import function pair — they are not interchangeable
- The untargeted path builds on pooled-QC-based filtering, so the patRoon data must include pooled quality control samples for the full GetFeatistics pipeline to be meaningful

## Evidence

- [abstract] Dedicated import functions for patRoon outputs: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL"
- [intro] Post-import pipeline context: "Once features have been extracted and potentially annotated, filtering based on QC quality cut-off criteria is typically performed"
- [readme] Package installation and loading: "devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE)"
- [readme] patRoon scope: [evidence span withheld — anchor status: not_found]
