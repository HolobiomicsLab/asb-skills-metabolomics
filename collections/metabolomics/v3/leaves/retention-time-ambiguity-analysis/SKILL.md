---
name: retention-time-ambiguity-analysis
description: Use when a metabolite standard or adduct in a metScribeR EIC review shows
  more than one manually labeled 'good' peak at different retention times, and ambiguous
  entries must be flagged or removed from the final RT/m/z library.
when_to_use_negative:
- Peaks have not yet been manually reviewed — the conflict check relies on the Good
  Peak / Bad Peak / Multimodal/Indeterminate labels assigned on the Review Results
  tab
- Each standard/adduct has only a single 'good' peak, so no competing RT assignment
  exists
- The goal is MS/MS-based identification — this skill operates on MS1 and RT data
  only, since metScribeR 'does not require MS/MS spectral information' for library
  construction
tools:
- name: metScribeR
  role: R package providing the find_adduct_conflicts check and the Shiny app whose
    Review Results and View/Export Library tabs hold the manually labeled peaks the
    conflict check consumes
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Language in which metScribeR is implemented and the conflict check is executed
- name: Shiny
  role: UI layer through which the 'check for crossed adducts' button is triggered
    on the View/Export Library tab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:6033ea73dd59e7845a585e0026d1b41ff4801f9adc837c3bf288285ac0c124ce
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# retention-time-ambiguity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies and removes 'crossed adducts' — metabolite standard adducts with multiple manually labeled 'good' peaks at divergent retention times — during metScribeR in-house LC-MS library construction, so that RT assignments retained in the final .csv library are unambiguous.

## When to use

Use when a metabolite standard or adduct in a metScribeR EIC review shows more than one manually labeled 'good' peak at different retention times, and ambiguous entries must be flagged or removed from the final RT/m/z library.

## When NOT to use

- Peaks have not yet been manually reviewed — the conflict check relies on the Good Peak / Bad Peak / Multimodal/Indeterminate labels assigned on the Review Results tab
- Each standard/adduct has only a single 'good' peak, so no competing RT assignment exists
- The goal is MS/MS-based identification — this skill operates on MS1 and RT data only, since metScribeR 'does not require MS/MS spectral information' for library construction

## Inputs

- metScribeR storage_object.RDS containing the peak-reviewed state of the library (manually labeled Good/Bad/Multimodal peaks)
- User-defined m/z and RT tolerance values (the same ones used for EIC creation)
- Standards/adducts derived from mzML files of authentic LC-MS standards (positive and negative ESI)

## Outputs

- crossed_adducts.csv listing standard identity, competing adducts, m/z, RTs, peak quality labels, and conflict metrics
- Updated/final library with ambiguous metabolite/adduct entries removed

## How to apply

Load the saved metScribeR storage_object.RDS (or storage_object_initial to skip recomputation) after peaks have been manually reviewed and labeled Good Peak / Bad Peak / Multimodal/Indeterminate. Run metScribeR::find_adduct_conflicts(storage_object, mz_tolerance, rt_tolerance) using the same user-defined m/z and RT tolerances that were set when creating the EICs, so adducts are distinguished consistently. The primary RT is taken from the highest-intensity manually labeled 'good' peak; any additional 'good' peak for the same standard that carries more than 50% of the primary apex intensity and differs in RT beyond the rt_tolerance is flagged as a crossed adduct. Collate all flagged entries into a table with standard identity, competing adducts, m/z, RTs, peak quality labels, and conflict metrics, write it to crossed_adducts.csv, and exclude these ambiguous metabolites/adducts from the exported library.

## Related tools

- **metScribeR** (R package providing the find_adduct_conflicts check and the Shiny app whose Review Results and View/Export Library tabs hold the manually labeled peaks the conflict check consumes) — https://github.com/ncats/metScribeR
- **R** (Language in which metScribeR is implemented and the conflict check is executed)
- **Shiny** (UI layer through which the 'check for crossed adducts' button is triggered on the View/Export Library tab)

## Examples

```
metScribeR::find_adduct_conflicts(storage_object, mz_tolerance, rt_tolerance)
```

## Evaluation signals

- crossed_adducts.csv exists, parses as a CSV, and contains identity, m/z, RT, and peak quality columns
- Every row corresponds to a peak pair whose RT separation exceeds the rt_tolerance used in the conflict check
- Each flagged competing peak's apex intensity exceeds 50% of the primary (highest-intensity 'good') peak's intensity
- The primary RT in each row is the highest-intensity manually labeled 'good' peak for that standard
- Flagged ambiguous entries are absent from the exported_metScribeR_library.csv

## Limitations

- Additional adducts from a standard are required to elute within the user-defined RT tolerance of the assigned RT; those beyond tolerance with substantial intensity are removed from the final library, reducing library coverage
- The conflict check is only as reliable as the manual peak review, since it depends on human Good/Bad/Multimodal labels
- Tolerances must match those used for EIC creation; inconsistent m/z/RT tolerance settings between steps change which peaks are considered competing adducts

## Evidence
