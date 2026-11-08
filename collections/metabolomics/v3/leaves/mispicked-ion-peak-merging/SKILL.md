---
name: mispicked-ion-peak-merging
description: Use when a Progenesis-format peak table from LC-MS/MS pre-processing
  shows split isotope/adduct features, and you need to identify or merge mispicked
  ions before downstream blank/replicate filtering.
when_to_use_negative:
- Input is raw vendor LC-MS/MS data not yet pre-processed into a Progenesis-format
  feature table — import_data() expects a peak table plus metadata.
- The goal is removing solvent blank carryover or non-reproducible features — those
  are handled by filter_group() and filter_cv(), not this filter.
- The target issue is in-source fragmentation rather than split isotopic patterns
  — use filter_insource_ions() instead.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mpactr
  role: Provides import_data(), filter_mispicked_ions() (with merge_peaks/merge_method
    arguments) and qc_summary() used to detect, merge, and report mispicked ions
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment in which the mpactr package (installed from CRAN or GitHub)
    is executed
  repo: https://github.com/mums2/mpactr/actions/workflows/r.yml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:372996b962548887d5ce6e4ce39670d5851dae46634a18697fd986cc8654aeea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mispicked-ion-peak-merging

## Summary

Detects and optionally merges mispicked MS1 features — isotopic patterns incorrectly split during pre-processing of tandem MS/MS data — using the mpactr filter_mispicked_ions() filter, and reports per-ion outcomes via qc_summary().

## When to use

Use when a Progenesis-format peak table from LC-MS/MS pre-processing shows split isotope/adduct features, and you need to identify or merge mispicked ions before downstream blank/replicate filtering.

## When NOT to use

- Input is raw vendor LC-MS/MS data not yet pre-processed into a Progenesis-format feature table — import_data() expects a peak table plus metadata.
- The goal is removing solvent blank carryover or non-reproducible features — those are handled by filter_group() and filter_cv(), not this filter.
- The target issue is in-source fragmentation rather than split isotopic patterns — use filter_insource_ions() instead.

## Inputs

- Progenesis-format peak table CSV (e.g. cultures_peak_table.csv)
- Progenesis-format metadata CSV (e.g. cultures_metadata.csv)
- mpactr object returned by import_data()

## Outputs

- Filtered mpactr object with mispicked ions corrected (peaks merged by summation)
- qc_summary() data.table with 'compounds' and 'status' columns ('passed' or failed-filter name)
- qc_summary.csv export file

## How to apply

Import the Progenesis-format peak table (e.g. cultures_peak_table.csv) and matching metadata (e.g. cultures_metadata.csv) with mpactr's import_data(), which returns an mpactr object. Run filter_mispicked_ions() with merge_peaks = TRUE and merge_method = 'sum' so that split isotopic signals are recombined by summing their abundances rather than only being flagged; the article notes this filter targets 'isotopic patterns that are incorrectly split during preprocessing'. All mpactr filters are independent, so this step can be placed first in a custom chain (e.g. followed by filter_group() on Solvent_Blank and Media groups and filter_cv() at cv_threshold = 0.2). After applying filters, call qc_summary() to obtain a data.table with one row per input compound id ('compounds' column) whose 'status' value is 'passed' or the name of the filter the ion failed (e.g. mispicked). Rationale: mispicked ions inflate feature counts and distort abundance measurements, so correcting them before blank and CV filtering yields a cleaner, high-quality feature table.

## Related tools

- **mpactr** (Provides import_data(), filter_mispicked_ions() (with merge_peaks/merge_method arguments) and qc_summary() used to detect, merge, and report mispicked ions) — https://github.com/mums2/mpactr
- **R** (Runtime environment in which the mpactr package (installed from CRAN or GitHub) is executed) — https://github.com/mums2/mpactr/actions/workflows/r.yml

## Examples

```
filtered <- mpactr::filter_mispicked_ions(import_data(peak_table, metadata), merge_peaks = TRUE, merge_method = "sum"); qc_summary(filtered)
```

## Evaluation signals

- qc_summary.csv exists and contains exactly one row per input compound with a non-empty 'compounds' field
- Every 'status' value is 'passed' or one of the applied filter names (mispicked, group, replicability) — no other values allowed
- With merge_peaks = TRUE and merge_method = 'sum', features previously split as isotopic patterns appear as single summed-abundance entries in the filtered peak table
- Mispicked ions removed by the filter are identifiable as rows whose qc_summary status names the 'mispicked' filter rather than 'passed'
- The filtered object retains metadata retrievable via get_metadata(), confirming import_data() inputs were consumed correctly

## Limitations

- Filters in mpactr are independent, so ordering is user-defined; running filter_mispicked_ions() after other filters changes which ions remain to be merged.
- The filter addresses only mispicked isotopic patterns from pre-processing; in-source fragment ions and blank carryover require separate filters.
- Filtering is defined on the feature/peak table level of pre-processed tandem MS/MS data, not on raw spectra.

## Evidence

- [methods] mispicked ions: isotopic patterns that are incorrectly split during preprocessing.: "mispicked ions: isotopic patterns that are incorrectly split during preprocessing."
- [methods] filter_mispicked_ions function call: "To check for mispicked ions, use mpactr function `filter_mispicked_ions()`"
- [readme] README filter description: [evidence span withheld — anchor status: external_doc]
- [other] qc_summary output description: [evidence span withheld — anchor status: not_found]
