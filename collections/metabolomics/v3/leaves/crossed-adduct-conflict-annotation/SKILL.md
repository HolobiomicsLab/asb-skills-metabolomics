---
name: crossed-adduct-conflict-annotation
description: Use when building an in-house LC-MS m/z-RT standard library in metScribeR
  and peaks have been filtered and manually reviewed — to detect conflicting adduct
  assignments and annotate each adduct with an identification probability prior to
  exporting the library CSV.
when_to_use_negative:
- Input is raw vendor or mzML data that has not yet been peak-picked, filtered, and
  manually reviewed — the crossed-adduct check operates on reviewed peak assignments
- MS2/MS/MS-centered library construction is the goal (e.g., GNPS-style spectral library
  building); metScribeR deliberately focuses on MS1 & RT data and MS2 is not required
  for library construction
- Already possess an exported library CSV with no changes to peak assignments or RT
  tolerances — rerunning the conflict check is unnecessary
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: metScribeR
  role: R/Shiny package whose 'check for crossed adducts' control (find_adduct_conflicts
    / update_best_peak_and_adduct) annotates adduct conflicts and identification probabilities
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: UI layer exposing the crossed-adduct check and library update/export buttons
  repo: https://github.com/ncats/metScribeR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:6a4024c7253fc8ae348987218ffe55836b01e0205fa58c015498dd1855863cea
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# crossed-adduct-conflict-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A metScribeR step that checks whether additional adducts of the same metabolite standard elute within a user-defined RT tolerance of the assigned RT, flags crossed/adduct conflicts, and attaches an internal identification probability for each adduct before library export.

## When to use

Use when building an in-house LC-MS m/z-RT standard library in metScribeR and peaks have been filtered and manually reviewed — to detect conflicting adduct assignments and annotate each adduct with an identification probability prior to exporting the library CSV.

## When NOT to use

- Input is raw vendor or mzML data that has not yet been peak-picked, filtered, and manually reviewed — the crossed-adduct check operates on reviewed peak assignments
- MS2/MS/MS-centered library construction is the goal (e.g., GNPS-style spectral library building); metScribeR deliberately focuses on MS1 & RT data and MS2 is not required for library construction
- Already possess an exported library CSV with no changes to peak assignments or RT tolerances — rerunning the conflict check is unnecessary

## Inputs

- standards_df.csv (common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path, optional inchiKey)
- adduct_df.csv (adduct, change_from_neutral, mode)
- mzML files for authentic standards in positive and negative ESI modes
- metScribeR storage_object.RDS with filtered, manually reviewed peak assignments

## Outputs

- updated in-app library table with identification probability per adduct and conflict annotations
- exported_metScribeR_library.csv (all 'good' adducts with identity, m/z, RT, peak quality metrics, manual quality label, identification probability)
- exported_metScribeR_library_with_metrics.csv (all good, bad, and multimodal/indeterminate adducts)
- storage_object.RDS capturing updated session state

## How to apply

After peaks pass filtering (≥7 data points per EIC, half-max width 0–15 s, intensity ≥10,000) and manual Good/Bad/Multimodal review, run the crossed-adduct check in metScribeR via find_adduct_conflicts and update_best_peak_and_adduct. The rule: additional adducts from a standard must elute within a user-defined RT tolerance of the assigned RT — pairs observed outside that window are treated as crossed/conflicting adducts. The check adds metScribeR's internal identification probability for each adduct (computed without requiring MS/MS spectral information); if desired, the 'search MoNA for MS/MS data' button can then supplement results using the inchiKey column of standards_df.csv. Remember to click the update library button first, since results do not appear in the displayed table otherwise, then export exported_metScribeR_library.csv with only 'good' adducts and save storage_object.RDS to preserve session state.

## Related tools

- **metScribeR** (R/Shiny package whose 'check for crossed adducts' control (find_adduct_conflicts / update_best_peak_and_adduct) annotates adduct conflicts and identification probabilities) — https://github.com/ncats/metScribeR
- **Shiny** (UI layer exposing the crossed-adduct check and library update/export buttons) — https://github.com/ncats/metScribeR

## Examples

```
devtools::install_github('ncats/metScribeR'); metScribeR::runMetScribeRShinyApp()
```

## Evaluation signals

- exported_metScribeR_library.csv exists and contains identity, m/z, RT, peak quality metrics, manual quality label, and MoNA MS/MS fields only for 'good' adducts
- Every adduct row carries an internal identification probability estimate (computed without MS/MS spectral information)
- Adducts assigned to the same standard but eluting outside the user-defined RT tolerance of the assigned RT are flagged as conflicts
- The in-app library table displays updated results only after the update library button has been pressed
- storage_object.RDS reloads to a session state consistent with the exported CSV

## Limitations

- Optional MoNA MS/MS retrieval depends on MoNA server speed and can often be slow
- RT tolerance distinguishing conflicting adducts is user-defined, so conflict calls are sensitive to that setting and LC-MS separation ability
- metScribeR does not require MS/MS spectral information, so identification probability estimates are internal probabilistic scores, not spectral-match confidences

## Evidence
