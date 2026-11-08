---
name: r6-reference-semantics-feature-table-filtering
description: Use when building or reviewing an mpactr pipeline where successive filters
  (filter_mispicked_ions, filter_cv, filter_insource_ions, filter_group) must either
  mutate the shared R6 mpactr object in place or leave the original peak table untouched
  for downstream QC and comparison.
when_to_use_negative:
- Input is already a deconvoluted feature table with no mpactr R6 object — this skill
  is about managing object-reference semantics during mpactr filter chains, not about
  importing generic CSVs.
- You are running a single one-off filter where you intend in-place mutation; reasoning
  about copy semantics adds no value there.
- The pipeline is written in Python using BalunasLab/mpact (the predecessor MPACT
  tool); the R6 copy_object semantics are specific to the R mpactr package.
tools:
- name: mpactr
  role: provides the R6 mpactr object and the copy_object flag on every filter function
    to control reference vs. copy semantics
  repo: https://github.com/mums2/mpactr
- name: R
  role: runtime environment; R6 class semantics are an R-language feature
  repo: https://github.com/mums2/mpactr/actions/workflows/r.yml
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:c0602b4b09846963a819b6117c51320150e2a18f65f9ce7bfb749738fdc32dcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - data.table
  - mpactr
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# R6 reference-semantics feature-table filtering

## Summary

mpactR stores the LC-MS/MS peak table, metadata, and filter status in an mpactr R6 object whose copy semantics are controlled per-filter by copy_object; understanding these reference semantics is required to keep pre-filter and post-filter feature tables distinct. Misusing them silently overwrites the original peak table.

## When to use

Use when building or reviewing an mpactr pipeline where successive filters (filter_mispicked_ions, filter_cv, filter_insource_ions, filter_group) must either mutate the shared R6 mpactr object in place or leave the original peak table untouched for downstream QC and comparison.

## When NOT to use

- Input is already a deconvoluted feature table with no mpactr R6 object — this skill is about managing object-reference semantics during mpactr filter chains, not about importing generic CSVs.
- You are running a single one-off filter where you intend in-place mutation; reasoning about copy semantics adds no value there.
- The pipeline is written in Python using BalunasLab/mpact (the predecessor MPACT tool); the R6 copy_object semantics are specific to the R mpactr package.

## Inputs

- Progenesis (or compatible tandem MS/MS) peak table
- sample metadata / combined sample list with biological-group and replicate labels
- user-chosen per-filter thresholds (e.g. cluster_threshold = 0.95 for in-source ions)
- per-filter copy_object flag (TRUE/FALSE)

## Outputs

- mpactr R6 object with updated filter state
- filtered peak table via mpactr::get_peak_table()
- metadata table via mpactr::get_metadata()
- filter summaries via filter_summary(..., 'insource') etc.

## How to apply

Create the mpactr R6 object once via mpactr::import_data() on the Progenesis peak table plus the combined sample metadata (e.g. the PTY087I2 dataset with 38 samples, solvent blanks, media blanks, and Streptomyces sp. PTY08712 grown ±250 µM cerium). Each filter call — filter_mispicked_ions(), filter_group(), filter_cv(), filter_insource_ions() — accepts a copy_object flag: set it TRUE to deep-copy the mpactr object so the original peak table and prior filter state stay untouched, or FALSE to mutate in place. Use copy_object=TRUE (as shown in the workflow: filter_insource_ions(mpactr_object, cluster_threshold = 0.95, copy_object = TRUE)) when you need to re-run a filter with different thresholds or compare pre-/post-filter feature counts; use FALSE to chain filters sequentially while keeping memory low. After filtering, call get_peak_table() to extract the filtered feature matrix and get_metadata() to verify sample–row alignment is intact. Verify the applied semantics by checking that the retained mpactr object reflects the last filter chain and that the pre-filter object (if kept) still reports the original ion count.

## Related tools

- **mpactr** (provides the R6 mpactr object and the copy_object flag on every filter function to control reference vs. copy semantics) — https://github.com/mums2/mpactr
- **R** (runtime environment; R6 class semantics are an R-language feature) — https://github.com/mums2/mpactr/actions/workflows/r.yml

## Examples

```
mpactr_obj <- mpactr::import_data(peak_table, metadata); filtered <- mpactr::filter_insource_ions(mpactr_obj, cluster_threshold = 0.95, copy_object = TRUE); ft <- mpactr::get_peak_table(filtered)
```

## Evaluation signals

- Sanity check: number of removed (failed) ions + passed ions after each filter equals the pre-filter input ion count for that filter.
- Confirm that get_peak_table() on a copy_object=TRUE chain returns a feature table whose row count differs from the original only by the number of ions flagged as failed by the filters.
- Verify that with copy_object=TRUE the original mpactr object still reports the original ion count when queried after filtering.
- Confirm metadata ordering from get_metadata() matches the peak table rows after filtering.
- Run a repeated pipeline with the same thresholds and confirm the same filtered feature count is reproduced.

## Limitations

- Filters are independent, so forgetting copy_object on a later filter can silently drop the earlier filter's effect from the retained object.
- copy_object=TRUE deep-copies the object, which increases memory use on large peak tables.
- The article/README does not document a default for copy_object, so a user must set it explicitly per call to get the semantics they want.

## Evidence

- [other] mpactR exposes in-source ion filtering as the exported function filter_insource_ions(mpactr_object, cluster_threshold, copy_object), which operates on the mpactr object and exposes a cluster_threshold argument: [evidence span withheld — anchor status: not_found]
- [other] Run mpactr::filter_insource_ions() on the mpactr object with cluster_threshold = 0.95 and copy_object = TRUE: [evidence span withheld — anchor status: not_found]
- [readme] Filters are independent, meaning they can be used to create a project-specific workflow: [evidence span withheld — anchor status: not_found]
- [methods] The goal of mpactr is to correct for errors that occur during the pre-processing of raw tandem MS/MS data: "The goal of mpactr is to correct for errors that occur during the pre-processing of raw tandem MS/MS data"
- [methods] To import these data into R, use the mpactr function import_data(): "To import these data into R, use the mpactr function `import_data()`"
