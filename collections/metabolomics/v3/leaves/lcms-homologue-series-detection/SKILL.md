---
name: lcms-homologue-series-detection
description: Use when a pre-processed LC-MS peak table (m/z, rt, intensity columns,
  e.g. the mtbls1358 dataset) should be annotated with homologue series IDs — e.g.
  to find polymer/PEG-like repeating-unit ladders or compare targeted vs untargeted
  detection runs.
when_to_use_negative:
- Input is raw LC-MS vendor data or an mzML file — the package operates on an already-peak-picked
  table with mz/rt/intensity columns
- The goal is general feature detection or peak picking rather than detecting repeating
  homologue-series ladders at a specific m/z increment
- Series of interest are shorter than min_series_length (e.g. < 5 members) or fall
  outside the supplied rt window (e.g. rt ≤ 500 s of the filtered mtbls1358 table)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: homologueDiscoverer
  role: Core R package providing detectHomologues (targeted/untargeted series detection),
    sdbCreate/sdbCheckContained/sdbPush/sdbSummarize (series database management),
    annotateHomologues, and plotting functions
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: tidyverse
  role: Loading and filtering the peak table (e.g. filter(., mz > 500, rt > 500))
    and manipulating annotated peak tables
- name: ggplot2
  role: Static visualization of annotated peak tables grouped by homologue_id via
    plotAnnotatedStatic
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:97e9f7b90ec675ead3cac0d8cd480bb63a4de992af6d1e44c7e38b6ffc42c2d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - detectHomologues
  - ggplot2
  - homologueDiscoverer
  - sdbCheckContained
  - sdbCreate
  - tidyverse
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# lcms-homologue-series-detection

## Summary

Detect homologue series in LC-MS metabolomics peak tables using the homologueDiscoverer R package, in either targeted mode (fixed m/z increment, e.g. 44.02628 Da) or untargeted mode (m/z increment window, e.g. 10–20), and manage/merge the resulting annotated series with a local series database.

## When to use

Use when a pre-processed LC-MS peak table (m/z, rt, intensity columns, e.g. the mtbls1358 dataset) should be annotated with homologue series IDs — e.g. to find polymer/PEG-like repeating-unit ladders or compare targeted vs untargeted detection runs.

## When NOT to use

- Input is raw LC-MS vendor data or an mzML file — the package operates on an already-peak-picked table with mz/rt/intensity columns
- The goal is general feature detection or peak picking rather than detecting repeating homologue-series ladders at a specific m/z increment
- Series of interest are shorter than min_series_length (e.g. < 5 members) or fall outside the supplied rt window (e.g. rt ≤ 500 s of the filtered mtbls1358 table)

## Inputs

- Pre-processed LC-MS peak table with mz, rt, and intensity columns (e.g. the mtbls1358 subset shipped in the homologueDiscoverer package)
- Detection parameters: search_mode (targeted/untargeted), step_mode (e.g. 'increment'), mz_min/mz_max or mz_steps, rt_min/rt_max, ppm_tolerance, min_series_length

## Outputs

- Annotated peak table with original mz, rt, intensity columns plus a homologue_id annotation column (e.g. annotated_peak_table_01.csv)
- Series database object (sdb) built via sdbCreate and extended via sdbPush
- Overlapping series identifiers from sdbCheckContained
- Summary object from sdbSummarize; static ggplot2 / interactive plots of annotated series

## How to apply

Load a peak table from the homologueDiscoverer package and filter it (e.g. mtbls1358 filtered to mz > 500 and rt > 500). Run detectHomologues in untargeted search_mode with an m/z increment window (mz_min = 10, mz_max = 20), rt constraints (rt_min = 1, rt_max = 100), ppm_tolerance = 5 and min_series_length = 5 using increment step_mode; then run detectHomologues in targeted search_mode with mz_steps = c(44.02628) under the same constraints. Internally the two-phase routine initializes 2-tuples from root peaks (initializeTargeted/initializeUntargeted) and extends them via expandChains under the ppm and relative rt tolerances, keeping series with at least min_series_length members. To combine runs, create a series database with sdbCreate, find overlapping series with sdbCheckContained (ppm_tolerance = 0.5, rt_tolerance = 0.5), push only non-overlapping series with sdbPush, and re-annotate the original table with annotateHomologues (ppm_matching_tolerance = 1, ppm_step_tolerance = 10). Summarize with sdbSummarize and visualize with plotAnnotatedStatic or ptbPlotInteractive.

## Related tools

- **homologueDiscoverer** (Core R package providing detectHomologues (targeted/untargeted series detection), sdbCreate/sdbCheckContained/sdbPush/sdbSummarize (series database management), annotateHomologues, and plotting functions) — https://github.com/kevinmildau/homologueDiscoverer
- **tidyverse** (Loading and filtering the peak table (e.g. filter(., mz > 500, rt > 500)) and manipulating annotated peak tables)
- **ggplot2** (Static visualization of annotated peak tables grouped by homologue_id via plotAnnotatedStatic)

## Examples

```
annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5, search_mode = "untargeted", step_mode = "increment")
```

## Evaluation signals

- Each output annotated peak table CSV exists and contains the original mz, rt, and intensity columns plus a homologue_id column
- Detected series respect the supplied ppm tolerance of 5 and the rt window of 1–100 seconds, and every series has length ≥ min_series_length (5)
- When comparing two runs, grouped-peak overlap (peaks sharing homologue_id membership) and exact-series correspondence counts are tabulated and discrepancies are quantified between them
- sdbCheckContained with ppm_tolerance = 0.5 and rt_tolerance = 0.5 correctly flags overlapping series so sdbPush adds only non-overlapping series
- sdbSummarize returns a summary of the merged series database covering both input runs

## Limitations

- Larger, more complex datasets (e.g. full MTBLS1358) show larger discrepancies between runs in grouped peaks and exact series correspondence than simple spiked data like PEG, making results sensitive to parameter settings (mz increment window, ppm_tolerance, rt constraints)
- Untargeted and targeted runs only partially agree; overlapping series must be reconciled via the series database (sdbCheckContained/sdbPush) rather than assumed identical
- The published tutorial uses only a subset of mtbls1358 (filtered to mz > 500, rt > 500); results are demonstrated on a limited example use case

## Evidence

- [abstract] homologueDiscoverer enables targeted and untargeted detection of homologue series in metabolomics data: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] Untargeted detection with m/z increment window 10–20 and rt 1–100: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, 
                                            rt_min = 1, rt_max = 100,"
- [methods] Targeted detection for the 44.02628 increment: "annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628),
                                            rt_min = 1, rt_max = 100,"
- [methods] Peak table load-and-filter step on mtbls1358: "peak_table <- mtbls1358 %>%
  filter(., mz > 500, rt > 500)"
- [methods] Series database reconciliation of overlapping runs: "overlapping_ids <- sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5)"
- [other] Comparison finding on MTBLS1358: discrepancies larger for grouped peaks than exact series correspondence: [evidence span withheld — anchor status: not_found]
- [other] Two-phase algorithm description from initialization to chain extension: [evidence span withheld — anchor status: not_found]
