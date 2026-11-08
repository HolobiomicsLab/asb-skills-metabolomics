---
name: metabolite-standard-library-quality-control
description: Use when building or curating an in-house LC-MS authentic-standard library
  from mzML files and you must set peak-filtering criteria, manually review peak quality,
  and check for crossed/unresolvable adducts before library export.
when_to_use_negative:
- Input is already a curated library table or feature table with no raw mzML chromatograms
  of authentic standards — this skill requires EIC peak extraction and per-peak manual
  review of the raw runs.
- Your identification strategy depends on MS/MS-based library construction — metScribeR
  focuses library building on MS1 & RT data, and MS2 data is not used or required
  for library construction.
- You need RTs transferable across laboratories or LC-MS conditions — RT is not yet
  consistently transferable or predictable across different LC-MS conditions; this
  skill only builds within-laboratory libraries.
tools:
- name: metScribeR
  role: R package implementing EIC extraction, peak filtering, manual peak quality
    review, crossed-adduct conflict checking, and CSV library export
  repo: https://github.com/ncats/metScribeR
- name: Shiny
  role: Web UI used to toggle density filter/data smoothing settings and click Good
    Peak / Bad Peak / Multimodal-Indeterminate labels during manual review
- name: R
  role: Runtime environment; metScribeR is installed via devtools and its find_adduct_conflicts
    function called on the storage object
- name: chromatographR
  role: GitHub dependency installed before metScribeR for chromatographic processing
  repo: https://github.com/ethanbass/chromatographR
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:25dc08a12e5b5d8e3dcb862debf5b7e0f3100bc494dcbe35ceb83d6c30b5f8ea
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolite-standard-library-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A QC workflow for in-house LC-MS metabolite reference libraries that filters poor-quality EIC peaks, obtains manual Good/Bad/Multimodal peak labels, and flags 'crossed adducts' whose additional 'good' peaks exceed 50% of the primary peak's apex intensity with RT differences beyond tolerance, removing them from the final library.

## When to use

Use when building or curating an in-house LC-MS authentic-standard library from mzML files and you must set peak-filtering criteria, manually review peak quality, and check for crossed/unresolvable adducts before library export.

## When NOT to use

- Input is already a curated library table or feature table with no raw mzML chromatograms of authentic standards — this skill requires EIC peak extraction and per-peak manual review of the raw runs.
- Your identification strategy depends on MS/MS-based library construction — metScribeR focuses library building on MS1 & RT data, and MS2 data is not used or required for library construction.
- You need RTs transferable across laboratories or LC-MS conditions — RT is not yet consistently transferable or predictable across different LC-MS conditions; this skill only builds within-laboratory libraries.

## Inputs

- standards_df.csv/tsv with required columns common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path (optional inchiKey, additional_identifiers)
- adduct_df.csv/tsv with required columns adduct, change_from_neutral, mode (POS or NEG)
- Positive- and negative-ESI LC-MS runs of authentic standards converted to .mzML format
- User-defined background noise level (below which all MS observations are excluded)
- User-defined m/z and RT tolerances for EIC creation and adduct distinguishability
- metScribeR storage_object.RDS or storage_object_initial RDS containing the peak-reviewed library state

## Outputs

- crossed_adducts.csv listing standard identity, competing adducts, m/z, RTs, peak quality labels, and conflict metrics
- adduct_conflicts.csv recording conflicting adduct identities and the tolerance criteria used
- exported_metScribeR_library.csv containing all 'good' adducts with identity, m/z, RT, peak quality information, and MoNA MS/MS spectra
- exported_metScribeR_library_with_metrics.csv including all 'good', 'bad', and 'multimodal/indeterminate' adducts with peak metrics
- A .png figure per manually reviewed peak in the Figures directory
- storage_object.RDS capturing the saved metScribeR state

## How to apply

Process each standard's positive- and negative-mode mzML files by extracting adduct EICs using user-defined m/z and RT tolerances chosen to match the LC-MS method's ability to confidently separate two signals. Apply the filtering criteria demonstrated on the IROA benchmark: at least 7 data points per peak in a single EIC, width at half maximum intensity greater than 0 s and less than 15 s, peak intensity at least 10,000 (10x the background noise level of 1000), and a practical maximum of 250 peaks per EIC. Then manually review every peak that passed filtering on the Review Results tab, labeling each Good Peak, Bad Peak, or Multimodal/Indeterminate; for each standard, the highest-intensity 'good' peak becomes the primary RT, and additional 'good' peaks are required to elute within the user-defined RT tolerance. Finally run find_adduct_conflicts(storage_object, mz_tolerance, rt_tolerance) with the same tolerances used for EIC creation: metabolites or adducts whose extra 'good' peaks carry more than 50% of the primary apex intensity and differ in RT beyond tolerance are flagged as crossed adducts and excluded from the final library, while remaining entries are checked pairwise for m/z and RT distinguishability.

## Related tools

- **metScribeR** (R package implementing EIC extraction, peak filtering, manual peak quality review, crossed-adduct conflict checking, and CSV library export) — https://github.com/ncats/metScribeR
- **Shiny** (Web UI used to toggle density filter/data smoothing settings and click Good Peak / Bad Peak / Multimodal-Indeterminate labels during manual review)
- **R** (Runtime environment; metScribeR is installed via devtools and its find_adduct_conflicts function called on the storage object)
- **chromatographR** (GitHub dependency installed before metScribeR for chromatographic processing) — https://github.com/ethanbass/chromatographR

## Examples

```
devtools::install_github('ncats/metScribeR'); library(metScribeR); runMetScribeRShinyApp()
```

## Evaluation signals

- crossed_adducts.csv exists, parses as a CSV, and contains identity, m/z, RT, and peak quality columns
- Every crossed-adduct row corresponds to a peak pair whose RT separation exceeds the rt_tolerance used in the conflict check
- No conflict is flagged for adduct pairs separable beyond both the specified m/z tolerance and RT tolerance
- exported_metScribeR_library.csv contains only 'good' adducts, while exported_metScribeR_library_with_metrics.csv additionally retains 'bad' and 'multimodal/indeterminate' entries
- On the IROA benchmark, metScribeR-derived RTs correlate with manual RTs at 0.99 and poor-quality peaks are filtered out

## Limitations

- Manual review of every filtered peak is required before library inclusion, and labeling a peak Multimodal/Indeterminate excludes it from the 'good' library.
- MoNA MS/MS retrieval is server-dependent and can be slow; MS/MS collection is optional and not required for the QC itself.
- Results do not appear in the displayed table until the update library button is pressed before export.
- Filtering criteria and conflict tolerances must be tuned per LC-MS method; the initial computation is slow for large submissions (~30 min for 12000 mzML files).

## Evidence
