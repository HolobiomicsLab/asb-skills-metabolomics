---
name: homologue-series-assignment
description: Use when two or more homologueDiscoverer detection runs (e.g. targeted
  mz_steps = c(44.02628) plus untargeted mz_min = 10 / mz_max = 20) share peaks and
  you must keep only non-overlapping series before pushing to the series database
  (sdb).
when_to_use_negative:
- The peak table has not yet been run through detectHomologues — greedy exclusion
  cannot be applied to raw, unannotated peaks.
- You intend to allow a peak to belong to multiple homologue series (e.g. exploratory
  co-series visualization with ptbPlotInteractive) rather than building an overlap-free
  database.
- The existing sdb is empty and there is no prior set of series to check containment
  against, making sdbCheckContained/sdbPush filtering moot.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: homologueDiscoverer
  role: 'Provides the full greedy-exclusion pipeline: detectHomologues to annotate
    peaks, sdbCreate/sdbCheckContained/sdbPush to build and filter the series database
    so each peak is assigned to exactly one series.'
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Execution environment; the greedy filtering is expressed as tidyverse dplyr
    filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids)) prior to
    sdbPush.
- name: tidyverse
  role: Supplies dplyr::filter used to exclude overlapping homologue_ids from the
    incoming annotated peak table before database insertion.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:2d187ecaaef95bef9dc7f3e3b23916e271160e7abf3f920594e9c9bafbef99d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - testthat
  - tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# homologue-series-assignment

## Summary

Greedy exclusion of homologue series detected by homologueDiscoverer so that each LC-MS peak is assigned to exactly one series, yielding overlap-free series databases.

## When to use

Use when two or more homologueDiscoverer detection runs (e.g. targeted mz_steps = c(44.02628) plus untargeted mz_min = 10 / mz_max = 20) share peaks and you must keep only non-overlapping series before pushing to the series database (sdb).

## When NOT to use

- The peak table has not yet been run through detectHomologues — greedy exclusion cannot be applied to raw, unannotated peaks.
- You intend to allow a peak to belong to multiple homologue series (e.g. exploratory co-series visualization with ptbPlotInteractive) rather than building an overlap-free database.
- The existing sdb is empty and there is no prior set of series to check containment against, making sdbCheckContained/sdbPush filtering moot.

## Inputs

- Filtered LC-MS peak table (tibble with mz, rt, intensity columns) from mtbls1358
- Existing series database created with sdbCreate
- Newly detected annotated peak table from detectHomologues (targeted or untargeted)

## Outputs

- Filtered annotated peak table excluding overlapping homologue_ids
- Updated local series database (sdb) where every peak is in at most one homologue series
- Summary of the series database via sdbSummarize

## How to apply

Run targeted and untargeted detectHomologues on the filtered mtbls1358 peak table (e.g. filter(mz > 500, rt > 500)), then call sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5) to retrieve the homologue_ids of the incoming set that overlap series already stored via sdbCreate/sdbPush. Greedily exclude those overlapping series with filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids)) before sdbPush, which removes peaks already assigned to a series from the search so every peak is annotated to belong to only one homologue series. Validate the final sdb with sdbSummarize and confirm no peak_id occurs in more than one series.

## Related tools

- **homologueDiscoverer** (Provides the full greedy-exclusion pipeline: detectHomologues to annotate peaks, sdbCreate/sdbCheckContained/sdbPush to build and filter the series database so each peak is assigned to exactly one series.) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Execution environment; the greedy filtering is expressed as tidyverse dplyr filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids)) prior to sdbPush.)
- **tidyverse** (Supplies dplyr::filter used to exclude overlapping homologue_ids from the incoming annotated peak table before database insertion.)

## Examples

```
sdb <- sdbPush(sdb, filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids)))
```

## Evaluation signals

- After sdbPush, sdbSummarize on the final sdb reports zero peaks assigned to more than one homologue series (no duplicated peak_id across series).
- sdbCheckContained on a re-insertion attempt of the excluded set returns non-empty overlapping_ids, confirming the greedy filter removed those series.
- The R script runs without error end-to-end (detectHomologues → sdbCreate → sdbCheckContained → filter → sdbPush → sdbSummarize).
- Number of series in the sdb equals the count of non-overlapping series retained after filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids)).
- annotateHomologues(ppm_matching_tolerance = 1, ppm_step_tolerance = 10) applied to the final sdb yields an annotated table with no peak matched to two homologue_ids.

## Limitations

- Greedy removal means a series that shares even one peak with an already-stored series is dropped entirely — homologue series sharing subsets of peaks are not split or merged.
- The ppm_tolerance = 0.5 and rt_tolerance = 0.5 thresholds in sdbCheckContained control overlap detection; peaks outside these windows may still be treated as non-overlapping.
- Greedy order dependence: the first series stored in the sdb claims its peaks, so the result can depend on which detection run (e.g. untargeted vs. targeted 44.02628) is loaded first.

## Evidence

- [discussion] greedy detection removes peaks already assigned to a homologue series: [evidence span withheld — anchor status: not_found]
- [methods] sdbCheckContained overlap retrieval: "overlapping_ids <- sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5)"
- [methods] greedy exclusion before sdbPush: "sdb <- sdbPush(sdb, filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids)))"
- [methods] initial sdb creation: "sdb <- sdbCreate(annotated_peak_table_01, sample_origin = "mtbls1358")"
- [methods] targeted detection run parameters: "annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628), rt_min = 1, rt_max = 100,"
