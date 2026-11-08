---
name: r6-reference-semantics-data-updating
description: Use when chaining multiple mpactr filters (mispicked, group, cv, insource)
  on one imported Progenesis peak table and you must decide whether each filter mutates
  the mpactR object in place or leaves the original intact via copy_object.
when_to_use_negative:
- When you need to branch a pipeline and run alternative filters from the same intermediate
  state — in-place reference updating (copy_object = FALSE) would mutate the shared
  object
- When the input is raw tandem MS/MS data rather than an already-imported mpactR object
  from a pre-processed Progenesis peak table — import_data() must run first
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mpactr
  role: Provides the filters (filter_mispicked_ions, filter_group, etc.) whose copy_object
    argument controls reference-semantics updating of the mpactR object
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment whose R6 reference semantics underlie the in-place vs
    copy behavior of mpactr filters
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:c9009226d9c067e35fd1985540623779bd445eb9a7567f712e3f2796c229047e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - data.table
  - mpactr
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# r6-reference-semantics-data-updating

## Summary

An R6/.reference-semantics skill for mpactr workflows: controlling whether each filter (e.g. filter_mispicked_ions with copy_object=FALSE) updates the mpactr object in place or operates on an independent copy, which determines how intermediate peak-table state is preserved across a multi-filter pipeline.

## When to use

Use when chaining multiple mpactr filters (mispicked, group, cv, insource) on one imported Progenesis peak table and you must decide whether each filter mutates the mpactR object in place or leaves the original intact via copy_object.

## When NOT to use

- When you need to branch a pipeline and run alternative filters from the same intermediate state — in-place reference updating (copy_object = FALSE) would mutate the shared object
- When the input is raw tandem MS/MS data rather than an already-imported mpactR object from a pre-processed Progenesis peak table — import_data() must run first

## Inputs

- mpactR object created by mpactr::import_data() from a Progenesis-format peak table (e.g. PTY087I2_dataset.csv) and sample metadata
- Filter-specific arguments including copy_object (TRUE for copy semantics, FALSE for in-place reference updating)

## Outputs

- Updated-in-place mpactR object (copy_object = FALSE) or an independent filtered copy (copy_object = TRUE)
- Filtered feature table via mpactr::get_peak_table()
- Filter outcome record via mpactr::filter_summary() (failed_ions and passed_ions lists)

## How to apply

After importing the Progenesis-format peak table and metadata with mpactr::import_data(), each mpactr filter exposes a copy_object argument that controls R6 reference semantics: with copy_object = FALSE the filter updates the mpactr object in place via reference, so subsequent calls (e.g. filter_group with group_threshold = 0.01 against 'Blanks' and 'Media', then filter_mispicked_ions with ringwin = 0.5, isowin = 0.01, trwin = 0.005, max_iso_shift = 3, merge_peaks = TRUE, merge_method = 'sum') see the accumulated filter state; with copy_object = TRUE the filter returns an independent copy, preserving the pre-filter object for comparison. Choose in-place updating when memory or object size matters and the sequential filter order is fixed; choose copying when you need to extract or compare peak tables at intermediate stages (e.g. pre- vs post-mispicked-merge via get_peak_table()). Regardless of mode, extract final results with get_peak_table() and audit outcomes with filter_summary(obj, 'mispicked') and get_similar_ions().

## Related tools

- **mpactr** (Provides the filters (filter_mispicked_ions, filter_group, etc.) whose copy_object argument controls reference-semantics updating of the mpactR object) — https://github.com/mums2/mpactr
- **R** (Runtime environment whose R6 reference semantics underlie the in-place vs copy behavior of mpactr filters)

## Examples

```
filter_mispicked_ions(data, ringwin=0.5, isowin=0.01, trwin=0.005, max_iso_shift=3, merge_peaks=TRUE, merge_method='sum', copy_object=TRUE)
```

## Evaluation signals

- With copy_object = TRUE, the original mpactR object remains unchanged after the filter call; with copy_object = FALSE, get_peak_table() on the same object reflects the applied filter
- Number of ions in the extracted peak table equals pre-filter count minus the failed (merged) ion count reported by filter_summary
- The filtered feature table and the get_similar_ions() record are non-empty data.table objects listing each retained main ion and its merged similar ions
- filter_summary(obj, 'mispicked') returns failed_ions and passed_ions lists consistent with the filter actually applied to the object state

## Limitations

- mpactr filters are independent by design, so the reference-vs-copy choice affects only object state management, not the filters' internal logic; filter order and project-specific workflow composition are the analyst's responsibility
- With copy_object = FALSE there is no automatic rollback — an incorrect filter parameter (e.g. an overly aggressive group_threshold) permanently mutates the in-place object

## Evidence

- [methods] with copy_object = FALSE the filter operates on reference semantics, updating the mpactr object in place rather than copying it: [evidence span withheld — anchor status: not_found]
- [readme] All filters are independent, meaning they can be used to create a project-specific workflow: "All filters are independent, meaning they can be used to create a
project-specific workflow"
- [methods] If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`: "If you want to extract the filtered peak table, with filters that have been applied, use `get_peak_table()`"
