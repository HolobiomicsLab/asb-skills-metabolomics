---
name: inchikey-based-spectral-lookup
description: Use when building an in-house m/z–RT metabolite standard library in metScribeR
  and the standards dataframe contains an inchiKey column, so public MS/MS spectra
  from MoNA should be merged into the exported library.
when_to_use_negative:
- The standards dataframe has no inchiKey column — MoNA lookup cannot be performed
  and MS/MS is optional since metScribeR does not require MS/MS data for library construction
- Input is raw vendor-format LC-MS data that has not yet been converted to mzML or
  processed through peak finding and manual review
- The goal is de novo MS/MS acquisition rather than supplementing the library with
  existing public spectra from MoNA
edam_operation: http://edamontology.org/operation_0405
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: metScribeR
  role: R/Shiny package whose MoNA search functionality queries MoNA's REST service
    via httr and exports the library and MS/MS CSVs
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: UI framework hosting the View/Export Library tab where the 'search MoNA for
    MS/MS data' and export buttons are triggered
- name: MassBank of North America (MoNA)
  role: Public spectral library searched by inchiKey as the source of retrieved MS/MS
    spectra
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:4866fd32e83a8568c70b5168016b906862be46a464c434721472145f630711e3
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
  - MassBank of North America (MoNA)
  - R
  - Shiny
  - metScribeR
  edam_operation: http://edamontology.org/operation_0405
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# inchikey-based-spectral-lookup

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augment an in-house LC-MS metabolite library by querying MoNA's REST service for MS/MS spectra using each standard's inchiKey column, retaining only fragment peaks above 40% relative abundance, and exporting the spectra alongside the library CSV and per-peak PNG reviews.

## When to use

Use when building an in-house m/z–RT metabolite standard library in metScribeR and the standards dataframe contains an inchiKey column, so public MS/MS spectra from MoNA should be merged into the exported library.

## When NOT to use

- The standards dataframe has no inchiKey column — MoNA lookup cannot be performed and MS/MS is optional since metScribeR does not require MS/MS data for library construction
- Input is raw vendor-format LC-MS data that has not yet been converted to mzML or processed through peak finding and manual review
- The goal is de novo MS/MS acquisition rather than supplementing the library with existing public spectra from MoNA

## Inputs

- metScribeR storage object (storage_object.RDS) from a previously started analysis
- standards dataframe (standards_df.csv/.tsv) with required columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path, and optional inchiKey
- InChIKey identifiers for each metabolite standard
- User-assigned manual peak quality labels (Good Peak / Bad Peak / Multimodal/Indeterminate) from the Review Results tab

## Outputs

- exported_metScribeR_library.csv containing identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields for all 'good' adducts
- exported_metScribeR_MoNA_MSMS.csv holding the retrieved MS/MS spectra (fragment peaks >40% relative abundance)
- exported_metScribeR_library_with_metrics.csv with all 'good', 'bad', and 'multimodal/indeterminate' adducts and peak metrics
- One .png per manually reviewed peak in the Figures directory
- Updated storage_object.RDS capturing session state

## How to apply

Load the saved metScribeR storage_object.RDS along with the standards dataframe whose inchiKey column links each standard to MassBank of North America. For every standard row with a non-missing InChIKey, run metScribeR's MoNA search (implemented with httr against the MoNA REST service) to retrieve MS/MS spectra — note this step depends on MoNA server speed and can be slow. Keep only fragment peaks with relative abundance above 40% of the maximum signal, then run the crossed-adduct check (find_adduct_conflicts / update_best_peak_and_adduct) to attach the internal identification probability for each adduct before export. Update the library, write exported_metScribeR_library.csv via export_library_csv and the retrieved spectra table via export_msms_csv to exported_metScribeR_MoNA_MSMS.csv, save a PNG per reviewed peak via peak_review_plot into the Figures directory, and persist session state with save_storage_object.

## Related tools

- **metScribeR** (R/Shiny package whose MoNA search functionality queries MoNA's REST service via httr and exports the library and MS/MS CSVs) — https://github.com/ncats/metScribeR
- **Shiny** (UI framework hosting the View/Export Library tab where the 'search MoNA for MS/MS data' and export buttons are triggered)
- **MassBank of North America (MoNA)** (Public spectral library searched by inchiKey as the source of retrieved MS/MS spectra)

## Examples

```
metScribeR::runMetScribeRShinyApp()  # then on View/Export Library tab: Update library -> 'search MoNA for MS/MS data' -> Export Library csv
```

## Evaluation signals

- exported_metScribeR_library.csv exists and contains identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields for all 'good' adducts
- exported_metScribeR_MoNA_MSMS.csv exists and holds retrieved spectra for every standard whose standards_df row had a non-missing inchiKey
- Retained fragment peaks in exported_metScribeR_MoNA_MSMS.csv all have relative abundance above 40% of the maximum signal
- A .png exists in the Figures directory for each manually reviewed peak
- storage_object.RDS reloads successfully so the analysis can resume without repeating the initial computation

## Limitations

- Collection of MS/MS information is dependent on the MoNA server speed and can often be slow
- Only peaks labeled 'good' appear in the primary exported_metScribeR_library.csv; 'bad' and 'multimodal/indeterminate' adducts are confined to the with_metrics file
- metScribeR focuses library building on MS1 & RT data; MS2 data is not used or required for library construction, so the MoNA lookup is an optional enrichment
- Missing or misspelled required column names in standards_df.csv (including inchiKey casing) will cause an error

## Evidence
