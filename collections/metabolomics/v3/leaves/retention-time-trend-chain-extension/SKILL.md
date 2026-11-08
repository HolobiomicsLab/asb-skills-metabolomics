---
name: retention-time-trend-chain-extension
description: Use when running untargeted homologue series detection on LC-MS peak
  tables and candidate series have reached three members, so their RT step trends
  must be learned to constrain further chain extension.
when_to_use_negative:
- When the homologue increment is already known and targeted detection is appropriate
  — use detectHomologues with mz_steps = c(44.02628) instead of letting expandChains
  learn RT trends from seed members
- When the input is already an annotated peak table with homologue_id assignments
  — the trend-constraint logic applies only during untargeted chain construction
- When features fall outside the mz > 500 / rt > 500 scope of the example pre-filtered
  peak table, trend extrapolation may behave differently than validated here
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: homologueDiscoverer
  role: R package providing detectHomologues (untargeted search_mode, step_mode =
    'increment') whose initializeUntargeted -> expandChains chain extension uses the
    first-three-member RT step trend to constrain subsequent search windows
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Runtime environment for executing the untargeted detection workflow and dplyr::filter
    pre-filtering
- name: tidyverse
  role: Provides dplyr::filter used to restrict the peak table to mz > 500 and rt
    > 500 before chain extension
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:48d49e99f4d96a1965a678737b7d703140ba578e77e446235580ad5a0931dffa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# retention-time-trend-chain-extension

## Summary

Constrains the extension of candidate homologue series in the untargeted detectHomologues algorithm (initializeUntargeted -> expandChains) of homologueDiscoverer by classifying the retention-time step trend of the first three members as mass-independent, increasing, or decreasing with mass, and using that trend to bound the search windows for subsequent members.

## When to use

Use when running untargeted homologue series detection on LC-MS peak tables and candidate series have reached three members, so their RT step trends must be learned to constrain further chain extension.

## When NOT to use

- When the homologue increment is already known and targeted detection is appropriate — use detectHomologues with mz_steps = c(44.02628) instead of letting expandChains learn RT trends from seed members
- When the input is already an annotated peak table with homologue_id assignments — the trend-constraint logic applies only during untargeted chain construction
- When features fall outside the mz > 500 / rt > 500 scope of the example pre-filtered peak table, trend extrapolation may behave differently than validated here

## Inputs

- LC-MS peak table (data.frame) loaded from the mtbls1358 example dataset bundled with homologueDiscoverer, with mz and rt columns
- detectHomologues untargeted parameter set: mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5, step_mode = 'increment'

## Outputs

- Annotated peak table (annotated_peak_table_01) with homologue_id assignments for each detected homologue series
- CSV export of the annotated peak table (e.g. annotated_peak_table_01.csv)
- Series usable for downstream ggplot2/plotAnnotatedStatic visualization and sdbCreate series databases

## How to apply

Load the mtbls1358-derived peak table from the homologueDiscoverer package and pre-filter features to m/z > 500 and retention time > 500 seconds with dplyr::filter, which reduces the search space before chain building. Call detectHomologues in untargeted search_mode with step_mode = 'increment', scanning mass differences between mz_min = 10 and mz_max = 20 (covering e.g. CH2 increments ~14 Da), an RT window of rt_min = 1 to rt_max = 100 seconds, ppm_tolerance = 5, and min_series_length = 5. Once a candidate chain reaches three members, assess the retention-time step trend across those members: mass-independent, increasing with mass, or decreasing with mass. Apply the classified trend to constrain the RT search window used to accept each subsequent member, so that extensions consistent with the observed trend are grown and inconsistent candidates are rejected. Write the returned annotated peak table, which carries homologue_id assignments per detected series, to CSV (e.g. annotated_peak_table_01.csv) for downstream visualization and series-database management.

## Related tools

- **homologueDiscoverer** (R package providing detectHomologues (untargeted search_mode, step_mode = 'increment') whose initializeUntargeted -> expandChains chain extension uses the first-three-member RT step trend to constrain subsequent search windows) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Runtime environment for executing the untargeted detection workflow and dplyr::filter pre-filtering)
- **tidyverse** (Provides dplyr::filter used to restrict the peak table to mz > 500 and rt > 500 before chain extension)

## Examples

```
annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5, search_mode = 'untargeted', step_mode = 'increment')
```

## Evaluation signals

- The annotated peak table contains homologue_id columns and each detected series has at least min_series_length = 5 members
- Detected series' within-series mass differences fall within the mz_min = 10 to mz_max = 20 scan window at ppm_tolerance = 5
- Member retention-time gaps stay within rt_min = 1 to rt_max = 100 seconds and are consistent with the trend classified from the first three members (mass-independent, increasing, or decreasing with mass)
- Each candidate series carries a coherent RT step trend (mass-independent, increasing with mass, or decreasing with mass) inferred from its first three members
- Exported annotated_peak_table_01.csv round-trips into sdbCreate and plotAnnotatedStatic without identifier collisions flagged by sdbCheckContained

## Limitations

- The trend constraint is only established once a candidate series reaches three members, so series shorter than three members cannot be trend-classified during extension
- Trend behavior was demonstrated on a subset of the mtbls1358 dataset filtered to mz > 500 and rt > 500; other mass/RT regimes may require different parameterization

## Evidence

- [abstract] Untargeted detection abstract claim: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] Untargeted detection call with scan window parameters: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100,"
- [methods] Peak table pre-filtering thresholds: "peak_table <- mtbls1358 %>%
  filter(., mz > 500, rt > 500)"
- [methods] Dataset source for the example run: "We make use of a subset of the mtbls1358 dataset for this example run."
- [other] Source-card finding on three-member trend classification constraining search windows: [evidence span withheld — anchor status: not_found]
