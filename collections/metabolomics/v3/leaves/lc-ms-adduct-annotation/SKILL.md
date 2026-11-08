---
name: lc-ms-adduct-annotation
description: Use when building an in-house LC-MS metabolite reference library from
  authentic-standard mzML data and retained adduct peaks (assigned m/z and RT) must
  be checked for distinguishability before export.
when_to_use_negative:
- Input is an already-annotated feature table from experimental (unknown) samples
  rather than authentic-standard mzML data — this skill builds the reference library,
  not downstream identifications
- The goal is de novo RT prediction transferable across different LC-MS methods —
  the article states RT is not yet consistently transferable across laboratory- or
  method-specific conditions
- The library must be built from MS/MS spectral data — metScribeR deliberately does
  not use or require MS/MS data for library construction
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3482
tools:
- name: metScribeR
  role: R package providing the Shiny workflow that extracts adduct chromatograms,
    performs peak finding, assesses peak quality, identifies conflicting adducts via
    the 'check for crossed adducts' step, and exports the library CSV
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Execution environment for loading the storage_object with readRDS and invoking
    the adduct conflict-checking function
- name: Shiny
  role: User interface through which the m/z and RT tolerances are supplied and the
    Review Results and View/Export Library tabs are driven
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:f7884b08396e881b42be4356c35b7cdb366737b9fea96ae626c2f1f11952c51c
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3482
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lc-ms-adduct-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Conflict-checks each retained adduct from authentic-standard LC-MS data against all other adducts in the library using user-defined m/z and RT tolerances, then annotates the exported library table. Ensures the final reference library contains only adduct peaks that can be confidently separated by the LC-MS method.

## When to use

Use when building an in-house LC-MS metabolite reference library from authentic-standard mzML data and retained adduct peaks (assigned m/z and RT) must be checked for distinguishability before export.

## When NOT to use

- Input is an already-annotated feature table from experimental (unknown) samples rather than authentic-standard mzML data — this skill builds the reference library, not downstream identifications
- The goal is de novo RT prediction transferable across different LC-MS methods — the article states RT is not yet consistently transferable across laboratory- or method-specific conditions
- The library must be built from MS/MS spectral data — metScribeR deliberately does not use or require MS/MS data for library construction

## Inputs

- metScribeR storage_object.RDS holding reviewed 'good' adduct peaks with assigned m/z and RT from authentic-standard mzML files
- adduct_df.csv (or .tsv) with columns adduct, change_from_neutral, mode (POS or NEG)
- standards_df.csv with common_name, monoisotopic_mass, and pos/neg_mode_mzML_file_path columns
- User-defined m/z tolerance and RT tolerance matched to the LC-MS method's signal-separation ability

## Outputs

- Conflict table (e.g. adduct_conflicts.csv) listing conflicting adduct pairs and the tolerance criteria used
- exported_metScribeR_library.csv with identity, m/z, RT, peak quality information, and manual peak quality labels
- exported_metScribeR_library_with_metrics.csv including all 'good', 'bad', and 'multimodal/indeterminate' adducts
- exported_metScribeR_MoNA_MSMS.csv with MS/MS information retrieved from MoNA

## How to apply

Load the processed metScribeR storage_object (containing reviewed 'good' adduct peaks with assigned m/z and RT) via readRDS from the prior peak-review step. Call the conflict-checking routine, e.g. metScribeR::find_adduct_conflicts(storage_object, mz_tolerance, rt_tolerance), where the user-defined m/z and RT tolerances are chosen to match the ability of the LC-MS equipment to confidently separate two signals. For each retained adduct, compare its m/z and retention time against every other adduct in the adduct library (defined by adduct_df.csv with adduct, change_from_neutral, and mode columns), flagging pairs as conflicting (crossed adducts) when they cannot be distinguished within both tolerance windows. Collect flagged pairs into a conflict table recording the conflicting adduct identities and tolerance criteria, and write it out (e.g. adduct_conflicts.csv). Finally, use the update library button and export the library CSV containing metabolite identity, m/z, RT, peak quality metrics, manual peak quality label, and optionally MoNA MS/MS data and internal identification probability estimates.

## Related tools

- **metScribeR** (R package providing the Shiny workflow that extracts adduct chromatograms, performs peak finding, assesses peak quality, identifies conflicting adducts via the 'check for crossed adducts' step, and exports the library CSV) — https://github.com/ncats/metScribeR
- **R** (Execution environment for loading the storage_object with readRDS and invoking the adduct conflict-checking function)
- **Shiny** (User interface through which the m/z and RT tolerances are supplied and the Review Results and View/Export Library tabs are driven)

## Examples

```
library(metScribeR); runMetScribeRShinyApp()  # then in the View/Export Library tab click 'check for crossed adducts' with user-defined m/z and RT tolerances
```

## Evaluation signals

- Every retained adduct in the storage_object was evaluated against all other adducts in the library (none skipped)
- Each reported conflict pair differs by less than both the specified m/z tolerance and RT tolerance; no conflict is flagged for pairs separable beyond those thresholds
- The final library CSV contains metabolite identity, m/z, RT, peak quality metrics, and manual quality labels for all 'good' adducts
- Manually derived RTs correlate ~0.99 with metScribeR-derived RTs (benchmark expectation), and poor-quality peaks are filtered out
- The update library button was pressed before viewing/exporting so results appear in the displayed table

## Limitations

- Tolerance values (m/z and RT) must be set based on the LC-MS equipment's ability to confidently separate two signals; mis-set tolerances cause false conflicts or missed cross-adducts
- MoNA MS/MS search depends on server speed and can often be slow
- RT values are reproducible within a single laboratory but not consistently transferable across LC-MS conditions
- Initial computation is slow for large submissions (~30 min for 12,000 mzML files); use the saved storage_object_initial to avoid repeating it

## Evidence
