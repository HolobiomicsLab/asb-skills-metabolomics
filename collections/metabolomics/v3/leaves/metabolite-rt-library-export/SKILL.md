---
name: metabolite-rt-library-export
description: Use when a reviewed metScribeR storage_object.RDS contains manually labeled
  peaks (good/bad/multimodal) and standards with InChIKeys, and the goal is to produce
  a final exported_metScribeR_library.csv plus MoNA MS/MS spectra and review figures.
when_to_use_negative:
- The workflow has not yet reached the Review Results stage — peaks must have passed
  filtering (≥7 data points per EIC, width at half maximum 0–15 s, intensity ≥10,000,
  ≤250 peaks per EIC) and been manually labeled before export is meaningful.
- You need MS/MS-centric library construction — metScribeR focuses on MS1 and RT data
  and does not require MS/MS spectral information for library building.
- The standards dataframe lacks correctly spelled inchiKey column values and you expect
  MoNA MS/MS retrieval — the MoNA search is triggered only for standards with an InChIKey.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: metScribeR
  role: R/Shiny package whose View/Export Library tab, update library button, crossed-adduct
    check, MoNA MS/MS search, export_library_csv / export_msms_csv functions, and
    storage_object mechanism perform the export
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Language in which metScribeR is implemented (uses httr for MoNA REST queries
    and devtools::install_github for installation)
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: UI framework hosting the View/Export Library tab where library update, MoNA
    search, and CSV export buttons are invoked
  repo: https://github.com/ncats/metScribeR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:64176abc92609b0757488ef6e1f5bb59015683d0933b45bc367183ce5e8c5e04
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - R
  - Shiny
  - metScribeR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-rt-library-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports a curated in-house LC-MS metabolite reference library from metScribeR, combining identity, m/z, RT, peak quality metrics, manual quality labels, MoNA-derived MS/MS spectra, and per-peak PNG reviews into CSV and figure outputs.

## When to use

Use when a reviewed metScribeR storage_object.RDS contains manually labeled peaks (good/bad/multimodal) and standards with InChIKeys, and the goal is to produce a final exported_metScribeR_library.csv plus MoNA MS/MS spectra and review figures.

## When NOT to use

- The workflow has not yet reached the Review Results stage — peaks must have passed filtering (≥7 data points per EIC, width at half maximum 0–15 s, intensity ≥10,000, ≤250 peaks per EIC) and been manually labeled before export is meaningful.
- You need MS/MS-centric library construction — metScribeR focuses on MS1 and RT data and does not require MS/MS spectral information for library building.
- The standards dataframe lacks correctly spelled inchiKey column values and you expect MoNA MS/MS retrieval — the MoNA search is triggered only for standards with an InChIKey.

## Inputs

- metScribeR storage_object.RDS (saved session state from a prior analysis)
- standards dataframe (standards_df.csv/.tsv) with common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path, and optional inchiKey column
- Prior manual peak quality labels (Good Peak / Bad Peak / Multimodal/Indeterminate) from the Review Results tab

## Outputs

- exported_metScribeR_library.csv (primary library of all 'good' adducts with identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields)
- exported_metScribeR_library_with_metrics.csv (all 'good', 'bad', and 'multimodal/indeterminate' adducts with peak metrics)
- exported_metScribeR_MoNA_MSMS.csv (all MS/MS information retrieved from MoNA)
- storage_object.RDS (updated session state)
- Figures directory containing one .png per manually reviewed peak

## How to apply

Load the previously saved metScribeR storage_object.RDS and the standards dataframe whose inchiKey column links each standard to MassBank of North America (MoNA). Click 'update library' so reviewed results appear in the displayed table — the README warns exports are invalid without this step. Optionally use the 'check for crossed adducts' and 'search MoNA for MS/MS data' buttons (httr-based REST queries against MoNA) to add internal identification probability per adduct and retrieve MS/MS spectra, retaining only fragment peaks with relative abundance above 40% of the maximum signal; note MoNA retrieval speed depends on the server and can be slow. Run export_library_csv to write exported_metScribeR_library.csv (identity, m/z, RT, peak quality metrics, manual quality label, MoNA MS/MS fields for all 'good' adducts) and export_msms_csv to write exported_metScribeR_MoNA_MSMS.csv, then save per-peak PNG visualizations into the Figures directory via peak_review_plot and persist state with save_storage_object. Validation confirms the library CSV contains identity, m/z, RT, quality metrics, manual labels, and MoNA MS/MS fields, that exported_metScribeR_MoNA_MSMS.csv holds the retrieved spectra, and that a .png exists in Figures for each reviewed peak.

## Related tools

- **metScribeR** (R/Shiny package whose View/Export Library tab, update library button, crossed-adduct check, MoNA MS/MS search, export_library_csv / export_msms_csv functions, and storage_object mechanism perform the export) — https://github.com/ncats/metScribeR
- **R** (Language in which metScribeR is implemented (uses httr for MoNA REST queries and devtools::install_github for installation)) — https://github.com/ncats/metScribeR
- **Shiny** (UI framework hosting the View/Export Library tab where library update, MoNA search, and CSV export buttons are invoked) — https://github.com/ncats/metScribeR

## Examples

```
library(metScribeR); runMetScribeRShinyApp()  # then in the View/Export Library tab: click 'update library', 'search MoNA for MS/MS data', and 'Export Library csv'
```

## Evaluation signals

- exported_metScribeR_library.csv exists and contains identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields for all 'good' adducts.
- exported_metScribeR_MoNA_MSMS.csv exists and holds the retrieved MoNA spectra (fragment peaks with relative abundance >40% of maximum signal for standards with InChIKeys).
- The Figures directory contains one .png file for every manually reviewed peak.
- storage_object.RDS reloads successfully so the session can be resumed without repeating the initial computation (~30 min for 12,000 mzML files).
- The displayed library table is non-empty only after the 'update library' button has been clicked — an empty table after export indicates the update step was skipped.

## Limitations

- Collection of MS/MS information depends on MoNA server speed and can often be slow.
- MoNA MS/MS retrieval is triggered only for standards rows containing an inchiKey; other identifiers in additional_identifiers are not used for spectral search.
- Incorrectly spelled or cased column names in standards_df.csv or adduct_df.csv (e.g. inchiKey, common_name, monoisotopic_mass) will cause an error.
- metScribeR only exports MS/MS spectra it retrieves from MoNA; it does not collect in-house MS/MS, so identification probability is reported per adduct without requiring spectral confirmation.

## Evidence

- [other] incorporates spectral peaks with relative abundance above 40% of the maximum signal into the exported library CSV.: [evidence span withheld — anchor status: not_found]
