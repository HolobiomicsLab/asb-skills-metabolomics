---
name: lc-ms-adduct-conflict-detection
description: Use when an in-house LC-MS standard library built with metScribeR contains
  metabolites or adducts with several 'good' labeled peaks at different RTs, and ambiguous
  entries must be flagged or excluded before exporting the library CSV.
when_to_use_negative:
- Input is raw mzML data that has not yet been through metScribeR EIC extraction,
  peak finding, filtering, and manual peak review — conflict detection operates on
  the reviewed peak state, not raw chromatograms
- MS2-dependent library construction is required — metScribeR focuses on MS1 and RT
  data and does not use MS/MS spectra for library construction
- A standard has only one labeled 'good' peak, or all competing peaks are below 50%
  of the primary peak's apex intensity or within the user-defined RT tolerance — no
  conflict exists to resolve
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metScribeR
  role: R package providing the find_adduct_conflicts function and the 'check for
    crossed adducts' button in the Shiny app that flags and removes ambiguous multi-peak
    adducts
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: UI layer through which the user sets m/z and RT tolerances and clicks 'check
    for crossed adducts' on the View/Export Library tab
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Runtime environment for loading storage_object.RDS and executing find_adduct_conflicts
    with the EIC tolerances
  repo: https://github.com/ncats/metScribeR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:e6b3d0ac5a1104232e53aa262689891b9188913f70ec663c938c94d84b4fbc10
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lc-ms-adduct-conflict-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects 'crossed adducts' in a metScribeR metabolite standard library: cases where a standard or adduct has multiple manually labeled 'good' peaks at distinct retention times. Ambiguous entries exceeding intensity/RT thresholds are flagged and removed from the final library.

## When to use

Use when an in-house LC-MS standard library built with metScribeR contains metabolites or adducts with several 'good' labeled peaks at different RTs, and ambiguous entries must be flagged or excluded before exporting the library CSV.

## When NOT to use

- Input is raw mzML data that has not yet been through metScribeR EIC extraction, peak finding, filtering, and manual peak review — conflict detection operates on the reviewed peak state, not raw chromatograms
- MS2-dependent library construction is required — metScribeR focuses on MS1 and RT data and does not use MS/MS spectra for library construction
- A standard has only one labeled 'good' peak, or all competing peaks are below 50% of the primary peak's apex intensity or within the user-defined RT tolerance — no conflict exists to resolve

## Inputs

- metScribeR storage_object.RDS containing the peak-reviewed library state (with manual Good Peak / Bad Peak / Multimodal labels)
- metScribeR storage_object_initial.RDS (optional, to restart without repeating the initial mzML loading computation)
- User-defined m/z and RT tolerances originally used for EIC creation and adduct distinction

## Outputs

- crossed_adducts.csv listing standard identity, competing adducts, m/z, RTs, peak quality labels, and conflict metrics
- Final exported_metScribeR_library.csv with ambiguous multi-peak adducts excluded

## How to apply

Load the saved metScribeR storage_object.RDS (or storage_object_initial to skip the initial ~30 min computation for large submissions) so the peak-reviewed library state is available. Run find_adduct_conflicts(storage_object, mz_tolerance, rt_tolerance) using the same user-defined m/z and RT tolerances set when creating EICs, so 'crossed adducts' are flagged where multiple manually labeled 'good' peaks compete for a standard. metScribeR selects the highest-intensity 'good' peak as the primary RT, then removes an adduct if any competing 'good' peak carries more than 50% of that apex intensity and its RT differs beyond the tolerance. Collate flagged entries into crossed_adducts.csv listing standard identity, competing adducts, m/z, RTs, peak quality labels, and conflict metrics, and exclude these ambiguous entries from the final library.

## Related tools

- **metScribeR** (R package providing the find_adduct_conflicts function and the 'check for crossed adducts' button in the Shiny app that flags and removes ambiguous multi-peak adducts) — https://github.com/ncats/metScribeR
- **Shiny** (UI layer through which the user sets m/z and RT tolerances and clicks 'check for crossed adducts' on the View/Export Library tab) — https://github.com/ncats/metScribeR
- **R** (Runtime environment for loading storage_object.RDS and executing find_adduct_conflicts with the EIC tolerances) — https://github.com/ncats/metScribeR

## Examples

```
library(metScribeR); runMetScribeRShinyApp()  # then on the View/Export Library tab click 'check for crossed adducts' before Export Library csv
```

## Evaluation signals

- crossed_adducts.csv exists, parses as a valid CSV, and contains identity, m/z, RT, and peak quality columns
- Every row corresponds to a peak pair whose RT separation exceeds the rt_tolerance used in the conflict check
- Every flagged competing 'good' peak has apex intensity greater than 50% of the primary peak's intensity — lower-intensity secondary peaks should not be flagged
- The number of reported ambiguous entries is consistent with the count of standards having multiple 'good' peaks in the reviewed library
- Excluded entries are absent from the final exported_metScribeR_library.csv but remain traceable in the metrics CSV (exported_metScribeR_library_with_metrics.csv)

## Limitations

- Conflict detection depends on the user-defined m/z and RT tolerances set at EIC creation; overly wide tolerances will merge distinct peaks while overly narrow tolerances will miss conflicts
- Detection requires prior manual review — only peaks already labeled 'good' via the Review Results tab (Good Peak / Bad Peak / Multimodal/Indeterminate) participate in conflict logic
- Removing crossed adducts reduces library coverage; the article/README does not provide a rescue mechanism, and MoNA MS/MS searches run slowly and are separate from this conflict step
- The manuscript did not state a specific metScribeR software version

## Evidence
