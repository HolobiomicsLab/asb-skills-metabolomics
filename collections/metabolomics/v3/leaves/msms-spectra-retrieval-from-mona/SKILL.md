---
name: msms-spectra-retrieval-from-mona
description: Use when a standards dataframe row contains an inchiKey (linking to MassBank
  of North America) and you want to enrich an exported LC-MS reference library with
  MoNA MS/MS spectra alongside the adduct RT/m/z and quality fields.
when_to_use_negative:
- The standards dataframe has no inchiKey column or rows with missing InChIKeys —
  MoNA retrieval only works for standards with an InChIKey.
- 'You want the MS/MS-free workflow: metScribeR does not require MS/MS spectral information
  and reports an identification probability estimate for each adduct without it.'
- Input is not in-house authentic-standard LC-MS mzML data processed through metScribeR's
  peak-finding and manual review steps.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metScribeR
  role: Provides the MoNA REST search (httr-based), crossed-adduct check (find_adduct_conflicts
    / update_best_peak_and_adduct), export_library_csv / export_msms_csv, and peak_review_plot
    functions executed from the Shiny app
  repo: https://github.com/ncats/metScribeR
- name: MassBank of North America (MoNA)
  role: External REST service queried with each standard's InChIKey to retrieve MS/MS
    spectra
- name: Shiny
  role: UI hosting the 'check for crossed adducts' and 'search MoNA for MS/MS data'
    buttons and the View/Export Library tab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:8fa02fe5468303a4f84e21dadf44221b6ddeb1d4058a1413dd86e234651b138f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# msms-spectra-retrieval-from-mona

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve MS/MS spectra for metabolite standards via MoNA's REST service using the inchiKey column of a metScribeR standards dataframe, retain fragment peaks with relative abundance above 40% of the maximum signal, and export them with the library. This adds orthogonal spectral evidence (identification probability) to the MS1-and-RT-focused metScribeR library workflow.

## When to use

Use when a standards dataframe row contains an inchiKey (linking to MassBank of North America) and you want to enrich an exported LC-MS reference library with MoNA MS/MS spectra alongside the adduct RT/m/z and quality fields.

## When NOT to use

- The standards dataframe has no inchiKey column or rows with missing InChIKeys — MoNA retrieval only works for standards with an InChIKey.
- You want the MS/MS-free workflow: metScribeR does not require MS/MS spectral information and reports an identification probability estimate for each adduct without it.
- Input is not in-house authentic-standard LC-MS mzML data processed through metScribeR's peak-finding and manual review steps.

## Inputs

- metScribeR storage object (storage_object.RDS) from a previously started analysis
- standards_df.csv with required columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path and optional inchiKey
- Processed peaks filtered per metScribeR criteria and manually labeled Good/Bad/Multimodal-Indeterminate in the Shiny app

## Outputs

- exported_metScribeR_library.csv with identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields for all 'good' adducts
- exported_metScribeR_MoNA_MSMS.csv holding the retrieved MS/MS spectra
- exported_metScribeR_library_with_metrics.csv including all 'good', 'bad', and 'multimodal/indeterminate' adducts
- Per-reviewed-peak PNG files in the Figures directory
- Updated storage_object.RDS session state

## How to apply

Load the saved metScribeR storage object (storage_object.RDS) and the standards dataframe whose inchiKey column links each standard to MoNA. For every standard with an InChIKey, query MoNA's REST service (metScribeR's MoNA search implemented with httr) and keep only fragment peaks whose relative abundance exceeds 40% of the maximum signal — an explicit spectral-quality cutoff. Before export, run the crossed-adduct check (find_adduct_conflicts / update_best_peak_and_adduct) so each adduct carries the internal identification probability. Then run update_library, write exported_metScribeR_library.csv (identity, m/z, RT, peak quality metrics, manual quality label, MoNA MS/MS fields) and exported_metScribeR_MoNA_MSMS.csv via export_msms_csv, save a per-peak PNG (peak_review_plot) to the Figures directory, and persist session state with save_storage_object. Expect slow collection: the README warns MS/MS collection 'is dependent on the MoNA server speed'.

## Related tools

- **metScribeR** (Provides the MoNA REST search (httr-based), crossed-adduct check (find_adduct_conflicts / update_best_peak_and_adduct), export_library_csv / export_msms_csv, and peak_review_plot functions executed from the Shiny app) — https://github.com/ncats/metScribeR
- **MassBank of North America (MoNA)** (External REST service queried with each standard's InChIKey to retrieve MS/MS spectra)
- **Shiny** (UI hosting the 'check for crossed adducts' and 'search MoNA for MS/MS data' buttons and the View/Export Library tab)

## Examples

```
devtools::install_github('ncats/metScribeR'); library(metScribeR); metScribeR::runMetScribeRShinyApp()
```

## Evaluation signals

- exported_metScribeR_MoNA_MSMS.csv exists and holds the retrieved spectra for standards with an InChIKey.
- exported_metScribeR_library.csv contains identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields for all 'good' adducts only.
- Exported spectral peaks are limited to relative abundance above 40% of the maximum signal.
- A .png exists in the Figures directory for each manually reviewed peak.
- storage_object.RDS reflects the post-export session state so the analysis can be resumed.

## Limitations

- Collection of MS/MS information depends on MoNA server speed and can often be slow (~30 min for 12000 mzML files was reported for the initial load; MoNA retrieval adds further latency).
- Only peaks with relative abundance above 40% of the maximum are incorporated, so low-abundance fragments are discarded from the exported spectra.
- Retrieval requires the standards_df to use exactly spelled column names, including inchiKey; other or incorrectly spelled/cased column names cause an error.
- MoNA searches only occur if explicitly requested via the 'search MoNA for MS/MS data' button — the base library workflow is MS1/RT only.

## Evidence
