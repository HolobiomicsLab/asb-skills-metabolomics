---
name: filter-summary-qc-accounting
description: Use when you have an mpactr mpactr object after running one or more filters
  on a Progenesis-format LC-MS/MS peak table and need to record which ions each filter
  removed or retained, and which mispicked ions were merged.
when_to_use_negative:
- 'Before any filter has been run: filter_summary(obj, filter) reports failed_ions/passed_ions
  for a specific applied filter step, not a global feature-quality verdict on the
  freshly imported object.'
- To obtain the filtered abundance matrix itself — use mpactr::get_peak_table() instead;
  filter_summary only returns ion lists, not intensities.
- When the input is a raw vendor peak table (e.g. Progenesis export) that has not
  yet been imported via mpactr::import_data() into an mpactr object.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mpactr
  role: R package whose filter_summary(obj, filter) returns failed_ions/passed_ions
    per filter and whose get_similar_ions() lists main ions merged during the mispicked-ions
    filter; used to extract the QC accounting from the mpactr object.
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment in which mpactr is executed; the QC summary is extracted
    as R data.table/list objects via the mpactr API.
  repo: https://github.com/mums2/mpactr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:2ae85d7caae28f5229233643876080e037f53ed89aa2deda998f658f56f1dad1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# filter-summary-qc-accounting

## Summary

Generates filter-level QC accounting for mpactr LC-MS/MS peak-table preprocessing, listing which ions failed or passed each filter step and surfacing groups of merged similar ions. This turns the output of any mpactr filter (e.g. filter_mispicked_ions, filter_group, filter_cv, filter_insource_ions) into an auditable record of ion counts and merge relationships.

## When to use

Use when you have an mpactr mpactr object after running one or more filters on a Progenesis-format LC-MS/MS peak table and need to record which ions each filter removed or retained, and which mispicked ions were merged.

## When NOT to use

- Before any filter has been run: filter_summary(obj, filter) reports failed_ions/passed_ions for a specific applied filter step, not a global feature-quality verdict on the freshly imported object.
- To obtain the filtered abundance matrix itself — use mpactr::get_peak_table() instead; filter_summary only returns ion lists, not intensities.
- When the input is a raw vendor peak table (e.g. Progenesis export) that has not yet been imported via mpactr::import_data() into an mpactr object.

## Inputs

- An mpactr mpactr object created by mpactr::import_data() from a Progenesis-format peak table and its sample metadata
- A quoted filter name string (e.g. 'mispicked') identifying the filter step to summarize
- For similar-ion accounting: the filtered mpactr object after filter_mispicked_ions(ringwin=0.5, isowin=0.01, trwin=0.005, max_iso_shift=3, merge_peaks=TRUE, merge_method='sum')

## Outputs

- failed_ions and passed_ions lists for the requested filter, from mpactr::filter_summary(obj, 'mispicked')
- A data.table listing each retained main ion and its merged similar ions, from mpactr::get_similar_ions()
- An ion-count reconciliation: filtered peak-table ion count = pre-filter ion count - failed ion count

## How to apply

After importing peak data with mpactr::import_data(peak_table, metadata, format='Progenesis') and applying any of the independent filters (filter_mispicked_ions, filter_group, filter_cv, filter_insource_ions), query the filtering outcome with mpactr::filter_summary(obj, filter) where filter is the quoted filter name, e.g. 'mispicked'. filter_summary returns the failed_ions and passed_ions lists for that step, giving an explicit ion-count record for QC. For the mispicked filter, pair this with mpactr::get_similar_ions() to retrieve a non-empty data.table listing each retained main ion and the similar ions merged into it (parameters ringwin=0.5, isowin=0.01, trwin=0.005, max_iso_shift=3, merge_peaks=TRUE, merge_method='sum'). Cross-check the accounting: the number of ions in the filtered feature table from get_peak_table() should equal the pre-filter ion count minus the count in failed_ions, and every merged similar ion should appear as a failed ion. Because filters can be run with copy_object=FALSE for reference semantics, summaries should be extracted before any further in-place mutation to keep the accounting tied to the intended filter state.

## Related tools

- **mpactr** (R package whose filter_summary(obj, filter) returns failed_ions/passed_ions per filter and whose get_similar_ions() lists main ions merged during the mispicked-ions filter; used to extract the QC accounting from the mpactr object.) — https://github.com/mums2/mpactr
- **R** (Runtime environment in which mpactr is executed; the QC summary is extracted as R data.table/list objects via the mpactr API.) — https://github.com/mums2/mpactr

## Examples

```
mpactr::filter_summary(data, 'mispicked'); mpactr::get_similar_ions(data)
```

## Evaluation signals

- filter_summary(obj, filter) returns non-empty failed_ions and passed_ions lists whose counts sum to the total ions entering that filter step.
- Reconciliation invariant: the ion count of the feature table returned by get_peak_table() equals the pre-filter ion count minus the number of ions in failed_ions for every filter that was applied.
- After filter_mispicked_ions(merge_peaks=TRUE), get_similar_ions() returns a non-empty data.table in which each similar (merged) ion maps to exactly one retained main ion.
- The merged similar ions listed by get_similar_ions() are a subset of the failed_ions returned by filter_summary for the 'mispicked' filter.
- If filters were applied with copy_object=FALSE (reference semantics), the mpactr object state at summary time must match the filter state being audited; re-extract summaries after any further in-place filtering.

## Limitations

- filter_summary is per-filter: it reports one filter's failed_ions/passed_ions at a time and does not aggregate across a multi-filter workflow, so each applied filter must be queried individually.
- With copy_object=FALSE the mpactr object is updated in place by reference semantics, so extracting the summary at the wrong time can record accounting for an unintended filter state.
- The similar-ions record is specific to the mispicked filter's merge behavior; other filters (group, cv, insource) have no analogous get_similar_ions() grouping record in the documented API.

## Evidence

- [methods] filter_summary returns failed_ions and passed_ions per filter step: [evidence span withheld — anchor status: not_found]
- [readme] readme documents the independent filters whose accounting is summarized: [evidence span withheld — anchor status: external_doc]
- [methods] copy_object=FALSE uses reference semantics, affecting when the QC summary is extracted: [evidence span withheld — anchor status: not_found]
- [methods] get_similar_ions returns the merged similar-ion record for the mispicked filter: [evidence span withheld — anchor status: not_found]
- [readme] mpactr's purpose is identifying high-quality MS1 features by correcting pre-processing peak-selection errors: [evidence span withheld — anchor status: external_doc]
