---
name: data-frame-structure-validation-against-mock-examples
description: Use when feature intensities and metadata have just been imported into
  R from a patRoon featureGroups/MFsummary object or an MS-DIAL area export, and you
  must confirm the resulting data frames match the expected featTable/featINFO schema
  before analysis.
when_to_use_negative:
- Input is already a validated GetFeatistics featTable/featINFO pair — re-running
  schema validation adds no information
- Data originates from a tool other than patRoon or MS-DIAL, so no dedicated import
  function and mock schema apply
- You are building custom plots or statistics from already-imported frames rather
  than checking import correctness
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: GetFeatistics
  role: Provides the import functions and the reference mock structures (df_example_feat_intensities,
    df_example_feat_info) that imported frames are validated against
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: patRoon
  role: Upstream non-target analysis tool whose featureGroups and MFsummary objects
    are converted to featTable/featINFO before validation
  repo: https://github.com/rickhelmus/patRoon
- name: MS-DIAL
  role: Upstream peak-detection tool whose area export is imported and then validated
    against the mock structures
- name: R
  role: Language in which the import and comparison of data frames is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:1bbc789b5b28ddc2950dba024392d808f22ed96977226a768ee18f589dc768a5
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# data-frame-structure-validation-against-mock-examples

## Summary

Validation practice from the GetFeatistics R-package: verify that feature tables and metadata imported from patRoon or MS-DIAL match the package's canonical mock structures df_example_feat_intensities and df_example_feat_info, so downstream QC filtering and statistics operate on correctly shaped data.

## When to use

Use when feature intensities and metadata have just been imported into R from a patRoon featureGroups/MFsummary object or an MS-DIAL area export, and you must confirm the resulting data frames match the expected featTable/featINFO schema before analysis.

## When NOT to use

- Input is already a validated GetFeatistics featTable/featINFO pair — re-running schema validation adds no information
- Data originates from a tool other than patRoon or MS-DIAL, so no dedicated import function and mock schema apply
- You are building custom plots or statistics from already-imported frames rather than checking import correctness

## Inputs

- patRoon featureGroups and MFsummary objects
- MS-DIAL area export (feature table export)
- Package mock examples df_example_feat_intensities and df_example_feat_info

## Outputs

- Validated featTable data frame (feature intensities, features x samples)
- Validated featINFO data frame (feature metadata)

## How to apply

After running the GetFeatistics import functions (get_feat_table_from_patRoon, get_feat_info_from_patRoon for patRoon featureGroups/MFsummary objects; get_feat_table_from_MSDial, get_feat_info_from_MSDial for MS-DIAL area exports), load the package's mock examples df_example_feat_intensities and df_example_feat_info as reference schemas. Compare the imported data frames against these mocks: feature-intensity frame (featTable) must have features as rows and samples as columns with numeric intensities, while the info frame (featINFO) must carry feature metadata fields with matching names and value types. Check that feature and sample identifiers align one-to-one between the two frames. Only when field names, types, and alignment match the mock structures should the data be passed on to pooled-QC filtration and downstream regression/statistical steps.

## Related tools

- **GetFeatistics** (Provides the import functions and the reference mock structures (df_example_feat_intensities, df_example_feat_info) that imported frames are validated against) — https://github.com/FrigerioGianfranco/GetFeatistics
- **patRoon** (Upstream non-target analysis tool whose featureGroups and MFsummary objects are converted to featTable/featINFO before validation) — https://github.com/rickhelmus/patRoon
- **MS-DIAL** (Upstream peak-detection tool whose area export is imported and then validated against the mock structures)
- **R** (Language in which the import and comparison of data frames is executed)

## Examples

```
devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE); library(GetFeatistics)
```

## Evaluation signals

- Imported featTable matches df_example_feat_intensities in column names and value types
- Imported featINFO matches df_example_feat_info in field names and types
- Feature and sample identifiers are aligned one-to-one across featTable and featINFO
- Intensity values are numeric and missing values are represented consistently with the mock structure
- Row counts correspond to the number of features in the source patRoon featureGroups or MS-DIAL export

## Limitations

- Validation only confirms schema (names, types, alignment), not correctness of intensity values from the upstream tool
- Only patRoon and MS-DIAL outputs have dedicated import functions and mock examples; other tools require manual schema construction
- Requires R >= 4.3.1 and Java of the same architecture (64-bit or 32-bit) as R for GetFeatistics to be usable

## Evidence

- [abstract] dedicated import functions: "For untargeted studies, it includes dedicated functions to import feature tables from tools like patRoon and MS-DIAL"
- [readme] README package purpose: "Getting streamlined elaboration of targeted and non-targeted metabolomics data, including elaboration of feature tables"
- [other] task_007 workflow validation step: [evidence span withheld — anchor status: not_found]
