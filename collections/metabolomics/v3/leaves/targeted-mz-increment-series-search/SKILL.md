---
name: targeted-mz-increment-series-search
description: Use when the goal is to find homologue series at a known m/z increment
  (e.g. 44.02628) in an LC-MS peak table with mz, rt, and intensity columns, and each
  detected peak should receive a homologue_id annotation.
when_to_use_negative:
- When the m/z increment is unknown and you want to discover all homologue series
  in a window — use untargeted 'increment' step mode with mz_min/mz_max (e.g. 10-20)
  instead
- When the peak table already carries homologue_id annotations or series are shorter
  than min_series_length, since those series will be silently dropped
- When the expected retention-time gaps between consecutive homologues fall outside
  the rt_min/rt_max window (1-100 s here), as chain extension will fail
tools:
- name: homologueDiscoverer
  role: R package providing detectHomologues in targeted 'increment' step mode, with
    internal initializeTargeted and expandChains routines for 2-tuple initialization
    and series extension under ppm and rt constraints
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Runtime environment in which detectHomologues is called on the filtered peak
    table
- name: tidyverse
  role: Used for filtering the mtbls1358 peak table (mz > 500, rt > 500) before targeted
    series detection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:a116f3ac48084486829ccadf16507229f916c5b0e5021410a4da23c9656c1a09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# targeted-mz-increment-series-search

## Summary

Detects homologue series in LC-MS peak tables when a specific m/z increment is known a priori, using homologueDiscoverer's detectHomologues in targeted 'increment' step mode. It initializes peak 2-tuples from the given increment and extends them under ppm and retention-time constraints.

## When to use

Use when the goal is to find homologue series at a known m/z increment (e.g. 44.02628) in an LC-MS peak table with mz, rt, and intensity columns, and each detected peak should receive a homologue_id annotation.

## When NOT to use

- When the m/z increment is unknown and you want to discover all homologue series in a window — use untargeted 'increment' step mode with mz_min/mz_max (e.g. 10-20) instead
- When the peak table already carries homologue_id annotations or series are shorter than min_series_length, since those series will be silently dropped
- When the expected retention-time gaps between consecutive homologues fall outside the rt_min/rt_max window (1-100 s here), as chain extension will fail

## Inputs

- LC-MS peak table (e.g. the mtbls1358 dataset subset bundled with homologueDiscoverer) with mz, rt, and intensity columns, filtered to mz > 500 and rt > 500 seconds
- One or more targeted m/z increment values (e.g. 44.02628)
- Constraint parameters: rt_min = 1, rt_max = 100 seconds, ppm_tolerance = 5, min_series_length = 5

## Outputs

- An annotated peak table (annotated_peak_table_02.csv) containing the original mz, rt, and intensity columns plus a homologue_id annotation column, where series respect the 5 ppm tolerance and 1-100 s rt window and have length >= 5

## How to apply

Load the peak table (e.g. the mtbls1358 subset shipped with homologueDiscoverer) and filter to m/z > 500 and rt > 500 seconds to focus on the homologue-relevant region. Call detectHomologues in targeted 'increment' step mode with mz_steps = c(44.02628), rt_min = 1, rt_max = 100, ppm_tolerance = 5, and min_series_length = 5. Internally, initializeTargeted builds 2-tuples from root peaks spaced by the supplied m/z increment, then expandChains screens the peak table for candidates matching the ppm tolerance (5 ppm here) and relative rt window (1-100 s) to extend each series. Keep only series with at least 5 members and annotate each peak of the filtered table with a homologue_id in the returned annotated peak table. Contrast this with the untargeted mode (mz_min = 10, mz_max = 20), which searches all increments in the window; the targeted mode restricts the search to increments you specify.

## Related tools

- **homologueDiscoverer** (R package providing detectHomologues in targeted 'increment' step mode, with internal initializeTargeted and expandChains routines for 2-tuple initialization and series extension under ppm and rt constraints) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Runtime environment in which detectHomologues is called on the filtered peak table)
- **tidyverse** (Used for filtering the mtbls1358 peak table (mz > 500, rt > 500) before targeted series detection)

## Examples

```
annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628), rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5)
```

## Evaluation signals

- The output CSV exists and contains the original mz, rt, and intensity columns plus a homologue_id annotation column
- Consecutive members of each detected series differ by the targeted increment (e.g. 44.02628) within the 5 ppm tolerance
- Retention-time differences between consecutive series members fall within the rt window of 1-100 seconds
- Every retained homologue series has at least min_series_length = 5 members
- The annotated peak table covers exactly the peaks of the input table filtered to mz > 500 and rt > 500

## Limitations

- The increment must be known a priori; targeting a wrong or single increment (e.g. only 44.02628) misses series with other spacing, so multiple mz_steps values may need to be supplied
- Series shorter than min_series_length are discarded, so genuine shorter homologues are lost
- Detection is sensitive to the ppm_tolerance (5) and rt window (1-100 s) choices; overly tight or loose tolerances break or over-extend chains
- Overlapping series from different runs (untargeted vs targeted) must be reconciled (e.g. via sdbCheckContained/sdbPush) before merging annotations

## Evidence

- [abstract] Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628), rt_min = 1, rt_max = 100,: "annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628), rt_min = 1, rt_max = 100,"
- [methods] The two-phase routine first checks for a second peak within the search window of a root peak to initialize 2-tuples, then uses the m/z increment and retention time constraints of each tuple to screen the peak table for candidates to extend the series.: [evidence span withheld — anchor status: not_found]
- [methods] We make use of a subset of the mtbls1358 dataset for this example run.: "We make use of a subset of the mtbls1358 dataset for this example run."
