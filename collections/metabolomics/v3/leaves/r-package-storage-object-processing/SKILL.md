---
name: r-package-storage-object-processing
description: Use when a previously started metScribeR analysis exists as a saved storage
  object (storage_object.RDS or storage_object_initial) and you need to continue peak
  review, run find_adduct_conflicts, or export the library without reprocessing mzML
  files.
when_to_use_negative:
- No prior metScribeR session exists — you must start from raw mzML files, standards_df.csv,
  and adduct_df.csv via the initial submit, not from a storage object
- The input is already a processed feature/peak table (e.g., from MZmine targeted
  feature detection) rather than the metScribeR state object, which carries only metScribeR's
  own workflow state
- The goal requires MS/MS-based library construction — metScribeR focuses on MS1 and
  RT data and 'MS2 data is not used or required for library construction'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: R package whose Shiny app persists and reloads the full analysis state as
    storage_object.RDS / storage_object_initial; exporting and state continuation
    functions come from this package
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: GitHub dependency installed before metScribeR, supporting the chromatographic
    peak handling behind the saved-state workflow
  repo: ethanbass/chromatographR
- name: R
  role: Runtime environment in which the storage object is read (readRDS) and the
    Shiny app is launched
- name: Shiny
  role: UI framework of the metScribeR app through which the saved state is loaded
    and tabs (Find Peaks, Review Results, View/Export Library) are driven
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:47cd09ca703ec5e2fb2183822a14096e46b232a07ae32e4d53b93cfebac858c0
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
  - chromatographR
  - metScribeR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-package-storage-object-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resume and continue a metScribeR in-house LC-MS metabolite library build by reloading the package's saved state object (storage_object.RDS or storage_object_initial) in R, so peak review, crossed-adduct checks, and library export proceed without repeating the long initial mzML computation.

## When to use

Use when a previously started metScribeR analysis exists as a saved storage object (storage_object.RDS or storage_object_initial) and you need to continue peak review, run find_adduct_conflicts, or export the library without reprocessing mzML files.

## When NOT to use

- No prior metScribeR session exists — you must start from raw mzML files, standards_df.csv, and adduct_df.csv via the initial submit, not from a storage object
- The input is already a processed feature/peak table (e.g., from MZmine targeted feature detection) rather than the metScribeR state object, which carries only metScribeR's own workflow state
- The goal requires MS/MS-based library construction — metScribeR focuses on MS1 and RT data and 'MS2 data is not used or required for library construction'

## Inputs

- storage_object.RDS — saved metScribeR state from a previous session (peak-reviewed library state)
- storage_object_initial — state captured after the initial submit finished loading mzML data
- Previously defined user parameters: background noise level, m/z tolerance, and RT tolerance used for EIC creation

## Outputs

- exported_metScribeR_library.csv — all 'good' adducts with identity, m/z, RT, peak quality info
- exported_metScribeR_library_with_metrics.csv — all good/bad/multimodal adducts with peak metrics
- exported_metScribeR_MoNA_MSMS.csv — MS/MS records retrieved from MoNA
- crossed_adducts.csv — flagged standards with competing adducts, m/z, RTs, quality labels, conflict metrics
- Updated storage_object.RDS and Figures directory of reviewed peak PNGs

## How to apply

Launch the metScribeR Shiny app in R (installed via devtools::install_github('ncats/metScribeR')) and load the saved storage object rather than re-submitting standards_df.csv/adduct_df.csv and raw mzML files, which take ~30 min for 12000 files on first submission. Use storage_object.RDS to resume from the last saved state (including manual Good Peak/Bad Peak/Multimodal review progress) or storage_object_initial to restart after the initial submit completed but before peak review. Continue on the Review Results and View/Export Library tabs, optionally running the 'check for crossed adducts' routine (find_adduct_conflicts with the same user-defined m/z and RT tolerances set when EICs were created) and the 'search MoNA for MS/MS data' step, then click 'update library' and 'Export Library csv'. The rationale is that the storage object captures the full data and workflow state, making large submissions resumable and reproducible across R sessions.

## Related tools

- **metScribeR** (R package whose Shiny app persists and reloads the full analysis state as storage_object.RDS / storage_object_initial; exporting and state continuation functions come from this package) — https://github.com/ncats/metScribeR
- **chromatographR** (GitHub dependency installed before metScribeR, supporting the chromatographic peak handling behind the saved-state workflow) — ethanbass/chromatographR
- **R** (Runtime environment in which the storage object is read (readRDS) and the Shiny app is launched)
- **Shiny** (UI framework of the metScribeR app through which the saved state is loaded and tabs (Find Peaks, Review Results, View/Export Library) are driven)

## Examples

```
library(metScribeR); obj <- readRDS('storage_object.RDS'); runMetScribeRShinyApp()
```

## Evaluation signals

- storage_object.RDS loads cleanly with readRDS in R and restores the workflow state so no re-computation of the ~30-min initial mzML submission is triggered
- After update + Export Library csv, exported_metScribeR_library.csv exists, parses as CSV, and contains identity, m/z, RT, and peak quality columns
- Every row in crossed_adducts.csv corresponds to a peak pair whose RT separation exceeds the rt_tolerance used in find_adduct_conflicts
- exported_metScribeR_library_with_metrics.csv includes all good, bad, and multimodal/indeterminate adducts, matching the manual review labels
- The Figures directory contains a .png for each manually reviewed peak, confirming the state was carried through review

## Limitations

- Initial submission of large studies is slow (~30 min for 12000 mzML files), which is precisely why state must be saved and reloaded
- MoNA MS/MS retrieval depends on server speed 'and can often be slow'
- The library table displays nothing until the 'update library' button is clicked; export only reflects saved state at the time of saving
- The metScribeR manuscript was 'currently in preparation' at README writing time; no version number is stated for the package

## Evidence
