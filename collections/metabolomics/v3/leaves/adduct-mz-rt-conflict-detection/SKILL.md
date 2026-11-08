---
name: adduct-mz-rt-conflict-detection
description: Use when finalizing an in-house LC-MS metabolite standard library built
  with metScribeR and you must flag adduct pairs whose m/z and RT overlap within the
  LC-MS method's separation ability before CSV export.
when_to_use_negative:
- The library CSV has already been exported and the goal is downstream identification
  of experimental compounds rather than library construction.
- Data are being peak-picked and annotated with external tools (MZmine, El-MAVEN,
  MS-DIAL) rather than processed through metScribeR's storage_object workflow.
- The task requires MS/MS spectral matching — metScribeR library building uses MS1
  and RT data and does not use MS2 for construction (MoNA spectra retrieval is a separate
  optional step).
tools:
- name: metScribeR
  role: R/Shiny package implementing the conflict check as the 'check for crossed
    adducts' step before library export
  repo: https://github.com/ncats/metScribeR
- name: R
  role: runtime environment in which metScribeR's storage_object is loaded with readRDS
    and tolerances are passed to the conflict check
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:ee2b534d0a402b5c56c93eca00784838544259cf7be01a75c4c018e8bc5e5970
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
  - metScribeR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# adduct-mz-rt-conflict-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

metScribeR's 'crossed adducts' conflict-checking step: each retained 'good' adduct from a metabolite standard is evaluated against every other adduct in the library, flagging pairs whose m/z and retention times cannot be distinguished within user-defined tolerances so only separable adducts enter the exported RT/m/z reference library.

## When to use

Use when finalizing an in-house LC-MS metabolite standard library built with metScribeR and you must flag adduct pairs whose m/z and RT overlap within the LC-MS method's separation ability before CSV export.

## When NOT to use

- The library CSV has already been exported and the goal is downstream identification of experimental compounds rather than library construction.
- Data are being peak-picked and annotated with external tools (MZmine, El-MAVEN, MS-DIAL) rather than processed through metScribeR's storage_object workflow.
- The task requires MS/MS spectral matching — metScribeR library building uses MS1 and RT data and does not use MS2 for construction (MoNA spectra retrieval is a separate optional step).

## Inputs

- storage_object.RDS (metScribeR state object with reviewed 'good' adduct peaks, assigned m/z and RT)
- User-defined m/z tolerance for distinguishing two adduct signals
- User-defined RT tolerance for distinguishing two adduct signals
- standards_df.csv and adduct_df.csv defining metabolite neutral masses and adduct mass shifts (POS/NEG mode)

## Outputs

- Conflict table of conflicting adduct pairs and the tolerance criteria used (e.g., adduct_conflicts.csv)
- exported_metScribeR_library.csv with metabolite identity, adduct, RT, m/z, peak quality metrics, and manual peak quality label
- exported_metScribeR_library_with_metrics.csv including 'good', 'bad', and 'multimodal/indeterminate' adducts

## How to apply

Load the processed storage_object.RDS (containing manually reviewed 'good' adduct peaks with assigned m/z and RT) from the prior metScribeR step with readRDS, or resume in the Shiny app on the View/Export Library tab. Choose an m/z tolerance and RT tolerance based on the LC-MS equipment's ability to confidently separate two signals, as the README instructs. Run the conflict check (functionally exposed in the app as the 'check for crossed adducts' button): for each retained adduct, compare its m/z and RT against every other adduct in the library, flagging pairs as conflicting when they cannot be distinguished within both tolerance windows. Confirm the library has been refreshed with the 'update library' button before checking, then export the library table — with metabolite identity, adduct, RT, m/z, and peak quality metrics — via the 'Export Library csv' button.

## Related tools

- **metScribeR** (R/Shiny package implementing the conflict check as the 'check for crossed adducts' step before library export) — https://github.com/ncats/metScribeR
- **R** (runtime environment in which metScribeR's storage_object is loaded with readRDS and tolerances are passed to the conflict check)

## Examples

```
storage_object <- readRDS('storage_object.RDS'); conflicts <- metScribeR::find_adduct_conflicts(storage_object, mz_tolerance, rt_tolerance); write.csv(conflicts, 'adduct_conflicts.csv')
```

## Evaluation signals

- Every retained 'good' adduct was evaluated against all other adducts in the library (no adduct skipped).
- Each reported conflict pair differs by less than both the specified m/z tolerance and the RT tolerance.
- No conflict is flagged for pairs separable beyond those tolerance thresholds.
- The displayed library table reflects the current state (library refreshed via the 'update library' button) before conflict checking.
- Exported CSV contains identity, adduct, RT, m/z, and peak quality columns required for identification workflows.

## Limitations

- m/z and RT tolerances must be set manually from the user's knowledge of the LC-MS method; overly generous tolerances over-flag conflicts, overly tight ones miss ambiguous adduct pairs.
- RT is not consistently transferable across different LC-MS conditions, so conflict judgments are method- and laboratory-specific.
- Checking must follow the manual Good Peak / Bad Peak / Multimodal/Indeterminate review; peaks mislabeled during review propagate into the conflict check.
- Conflict checking operates only on retained 'good' adducts; poor-quality peaks removed by the filters (e.g., <7 data points per EIC, intensity <10,000) are not part of the check.

## Evidence

- [readme] choose an m/z and RT tolerance for creating EICs and distinguishing between adducts. These tolerances should be set based on the ability for the LC-MS equipment to confidently separate two signals: "choose an m/z and RT tolerance for creating EICs and distinguishing between adducts. These tolerances should be set based on the ability for the LC-MS equipment to confidently separate two signals."
