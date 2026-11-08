---
name: peak-grouping-overlap-quantification
description: Use when two homologue-series annotation runs over the same peak table
  (e.g. untargeted vs targeted detectHomologues, or HD vs nontarget) must be compared
  for grouped-peak overlap and exact-series correspondence, especially on complex
  datasets like MTBLS1358.
when_to_use_negative:
- Input is raw LC-MS spectra rather than a peak/feature table with mz, rt and intensity
  columns
- Only one homologue-series annotation exists — there is no second run to quantify
  overlap against
- The dataset is a simple homopolymer standard (e.g. PEG spiked data) where grouped
  peaks already overlap substantially and the discrepancy analysis adds little
tools:
- name: homologueDiscoverer
  role: R package providing the annotated peak tables (detectHomologues untargeted
    and targeted runs) and the mtbls1358 peak table being compared
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: detectHomologues
  role: Generates the two annotation runs whose grouped-peak overlap and exact-series
    correspondence are quantified
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: sdbCheckContained
  role: Establishes exact-series correspondence by identifying overlapping series
    identifiers between runs at ppm_tolerance = 0.5 and rt_tolerance = 0.5
- name: tidyverse
  role: Filtering the peak table and merging/counting homologue_id membership across
    the two annotated tables
- name: ggplot2
  role: Visualization of annotated peak tables grouped by homologue_id to inspect
    overlap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:756339155e0f016dacf6b252ec29d2810f5787fde92d13cf866e1f8c4ff1af67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - detectHomologues
  - ggplot2
  - homologueDiscoverer
  - sdbCheckContained
  - sdbCreate
  - tidyverse
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# peak-grouping-overlap-quantification

## Summary

Quantifies agreement between two homologue series annotation runs (e.g. homologueDiscoverer untargeted vs targeted, or HD vs nontarget) on an LC-MS peak table by counting grouped-peak overlap (peaks sharing homologue_id membership) and exact-series correspondence (matched homologue series). This exposes where detector disagreement concentrates — grouped peaks vs whole series.

## When to use

Use when two homologue-series annotation runs over the same peak table (e.g. untargeted vs targeted detectHomologues, or HD vs nontarget) must be compared for grouped-peak overlap and exact-series correspondence, especially on complex datasets like MTBLS1358.

## When NOT to use

- Input is raw LC-MS spectra rather than a peak/feature table with mz, rt and intensity columns
- Only one homologue-series annotation exists — there is no second run to quantify overlap against
- The dataset is a simple homopolymer standard (e.g. PEG spiked data) where grouped peaks already overlap substantially and the discrepancy analysis adds little

## Inputs

- mtbls1358 peak table from the homologueDiscoverer package (data frame with mz, rt and intensity columns)
- Annotated peak table from run 1 (e.g. untargeted detectHomologues output with homologue_id column)
- Annotated peak table from run 2 (e.g. targeted detectHomologues output or nontarget output with homologue_id column)
- Matching tolerances for series identity checks (ppm_tolerance, rt_tolerance)

## Outputs

- Merged annotated peak table with homologue_id membership from both runs
- Comparison table with grouped-peak overlap counts and exact-series correspondence counts
- homologue_comparison.csv

## How to apply

Load the peak table (e.g. the mtbls1358 subset shipped with the homologueDiscoverer package) and filter to the region of interest, e.g. mz > 500 and rt > 500. Produce both annotations to be compared — for instance an untargeted detectHomologues run (mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5, increment step_mode) and a targeted run (mz_steps = c(44.02628), same rt/ppm/min_series_length settings) — or an HD run paired with an nontarget (NT) run. Merge the two annotated peak tables and compute two metrics: grouped-peak overlap, counting peaks that share homologue_id membership across runs, and exact-series correspondence, counting matched homologue series between runs (series identity can be established with sdbCheckContained at ppm_tolerance = 0.5 and rt_tolerance = 0.5). Tabulate both metrics side by side and explicitly test whether discrepancies are larger at the peak-grouping level than at the whole-series level; the article's MTBLS1358 comparison showed larger discrepancies in grouped peaks than in exact series correspondence, unlike PEG spiked data where grouped peaks overlapped substantially. Write the comparison table to homologue_comparison.csv.

## Related tools

- **homologueDiscoverer** (R package providing the annotated peak tables (detectHomologues untargeted and targeted runs) and the mtbls1358 peak table being compared) — https://github.com/kevinmildau/homologueDiscoverer
- **detectHomologues** (Generates the two annotation runs whose grouped-peak overlap and exact-series correspondence are quantified) — https://github.com/kevinmildau/homologueDiscoverer
- **sdbCheckContained** (Establishes exact-series correspondence by identifying overlapping series identifiers between runs at ppm_tolerance = 0.5 and rt_tolerance = 0.5)
- **tidyverse** (Filtering the peak table and merging/counting homologue_id membership across the two annotated tables)
- **ggplot2** (Visualization of annotated peak tables grouped by homologue_id to inspect overlap)

## Examples

```
annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5, step_mode = "increment")
```

## Evaluation signals

- homologue_comparison.csv exists and contains columns for grouped-peak overlap and exact-series correspondence counts
- Reported discrepancies are larger for grouped peaks than for exact-series correspondence (the expected MTBLS1358 pattern)
- Series identity matches are confirmed via sdbCheckContained with ppm_tolerance = 0.5 and rt_tolerance = 0.5
- Both input annotation tables were produced with consistent detection settings (rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5)
- Grouped-peak overlap counts are reproducible from the merged table by recomputing shared homologue_id memberships

## Limitations

- On the more complex MTBLS1358 dataset, HD and NT showed larger discrepancies in both grouped peaks and exact series correspondence than on PEG spiked data, so conclusions are dataset-dependent
- Results are sensitive to search settings (mz_min, mz_max, rt window, ppm_tolerance, min_series_length, step_mode), which must be fixed and reported when comparing runs

## Evidence

- [abstract] homologueDiscoverer enables targeted and untargeted detection of homologue series: "allows for the targeted and untargeted detection of homologue series as well as their evaluation and management using interactive plots and simple local database functionalities"
- [methods] MTBLS1358 peak table load and filter: "peak_table <- mtbls1358 %>%
  filter(., mz > 500, rt > 500)"
- [methods] Overlap detection between two annotation runs: "overlapping_ids <- sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5)"
- [methods] Example dataset used for the comparison run: "We make use of a subset of the mtbls1358 dataset for this example run."
- [results] Dataset sensitivity of the comparison: [evidence span withheld — anchor status: not_found]
