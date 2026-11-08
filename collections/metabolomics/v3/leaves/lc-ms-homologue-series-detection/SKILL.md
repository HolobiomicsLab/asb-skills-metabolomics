---
name: lc-ms-homologue-series-detection
description: Use when given an LC-MS feature/peak table (mz, rt, intensity) and the
  goal is to group features into homologue series defined by repeating mass increments
  (e.g. 44.02628 Da C2H4O) with correlated retention-time steps.
when_to_use_negative:
- Input is raw LC-MS vendor data or mzML centroids that still need feature detection/alignment
  into a peak table — homologueDiscoverer expects a pre-built peak table with mz,
  rt, intensity.
- Input is already annotated with homologue_id values from a prior run and no new
  series detection is needed.
- Features lack any repeating mass increment or RT-trend structure (e.g. single-class
  spectra without homologues).
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: homologueDiscoverer
  role: R package providing detectHomologues (targeted/untargeted series detection),
    annotateHomologues (peak-level assignment from sdb), sdbCreate/sdbPush/sdbCheckContained/sdbSummarize
    (series database management), and plotting functions
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: tidyverse
  role: Used for filtering peak tables (e.g. filter(., mz > 500, rt > 500)) and overlap
    counting logic
- name: ggplot2
  role: Underlies plotAnnotatedStatic visualization of annotated peak tables grouped
    by homologue_id
- name: RColorBrewer
  role: Provides color palette for ggplot2 homologue_id visualization via getPalette
    = colorRampPalette(RColorBrewer::brewer.pal(9, "Set1"))
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_007
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:2a8218c5736f0db21b2a9d11ebea9a4bb52ac38b3761d6eeb75dd29f1a668f1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - RColorBrewer
  - ggplot2
  - homologueDiscoverer
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-ms-homologue-series-detection

## Summary

An LC-MS metabolomics skill for detecting homologue series (repeating-unit mass/RT trends across feature groups) in peak tables, using the R package homologueDiscoverer for targeted and untargeted detection, series management in a local series database (sdb), and peak-level annotation.

## When to use

Use when given an LC-MS feature/peak table (mz, rt, intensity) and the goal is to group features into homologue series defined by repeating mass increments (e.g. 44.02628 Da C2H4O) with correlated retention-time steps.

## When NOT to use

- Input is raw LC-MS vendor data or mzML centroids that still need feature detection/alignment into a peak table — homologueDiscoverer expects a pre-built peak table with mz, rt, intensity.
- Input is already annotated with homologue_id values from a prior run and no new series detection is needed.
- Features lack any repeating mass increment or RT-trend structure (e.g. single-class spectra without homologues).

## Inputs

- LC-MS feature/peak table with peak_id, mz, rt, intensity columns (e.g. CSV/TSV)
- Bundled example peak table such as mtbls1358 from the homologueDiscoverer package
- Known repeating-unit mass (mz_steps, e.g. 44.02628 for C2H4O) for targeted mode

## Outputs

- Annotated peak table adding homologue_id per peak (e.g. annotated_peak_table_01.csv / annotated_peaks.csv)
- Series database (sdb) object created via sdbCreate with sample_origin metadata and extended via sdbPush
- Overlap report from sdbCheckContained listing overlapping series identifiers
- sdbSummarize summary of stored series
- Static ggplot/interactive Shiny plots (plotAnnotatedStatic / ptbPlotInteractive)

## How to apply

Load a peak table with peak_id, mz, rt, intensity columns and, for exploratory runs, pre-filter features (e.g. mtbls1358 subset: mz > 500 and rt > 500). Run detectHomologues in untargeted search_mode with step_mode = 'increment' scanning mass differences in a window (e.g. mz_min = 10 to mz_max = 20) constrained by a retention-time window (e.g. rt_min = 1, rt_max = 100 seconds), ppm tolerance (e.g. 5 ppm), and a minimum series length (e.g. 5 members); for known repeating units, run a second detectHomologues pass in targeted search_mode with explicit mz_steps such as c(44.02628). Before merging multiple annotated tables, check overlap with sdbCheckContained (ppm_tolerance = 0.5, rt_tolerance = 0.5) and only sdbPush series whose homologue_id is not already contained in the sdb series database; finalize peak-level annotation with annotateHomologues (ppm_matching_tolerance = 1, ppm_step_tolerance = 10, rt_tolerance = 1, min_match_length = 4) and summarize with sdbSummarize. Visualize and curate with plotAnnotatedStatic or ptbPlotInteractive, which require a Shiny/vignette-style runtime. Decision point: series extension trends — the first three members' RT steps are classified as mass-independent, increasing, or decreasing with mass, which constrains subsequent search windows.

## Related tools

- **homologueDiscoverer** (R package providing detectHomologues (targeted/untargeted series detection), annotateHomologues (peak-level assignment from sdb), sdbCreate/sdbPush/sdbCheckContained/sdbSummarize (series database management), and plotting functions) — https://github.com/kevinmildau/homologueDiscoverer
- **tidyverse** (Used for filtering peak tables (e.g. filter(., mz > 500, rt > 500)) and overlap counting logic)
- **ggplot2** (Underlies plotAnnotatedStatic visualization of annotated peak tables grouped by homologue_id)
- **RColorBrewer** (Provides color palette for ggplot2 homologue_id visualization via getPalette = colorRampPalette(RColorBrewer::brewer.pal(9, "Set1")))

## Examples

```
annotated <- homologueDiscoverer::detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5)
```

## Evaluation signals

- Peak-table format check: detectHomologues/checkPeakTableFormat expects columns peak_id, mz, rt, intensity.
- Series length invariant: no output series should have fewer members than min_series_length (e.g. 5).
- Expected range benchmark: on the PEG17 spiked plasma dataset (276 peaks) exactly 115 peaks are annotated to a homologue series; on PEG70 (251 peaks) exactly 140.
- Mass-window invariant: untargeted runs must restrict candidate mass differences to the mz_min–mz_max window (e.g. 10–20 Da) and RT steps to rt_min–rt_max (e.g. 1–100 s).
- Overlap handling: after sdbCheckContained with ppm_tolerance = 0.5 and rt_tolerance = 0.5, sdbPush must be filtered to exclude overlapping homologue_ids (filter(annotated_peak_table_02, !(homologue_id %in% overlapping_ids))).

## Limitations

- homologueDiscoverer (HD) and nontarget (NT) show large discrepancies in exact series overlap (identical peak members in the same order) even when grouped feature peaks overlap substantially — cross-tool comparisons should not assume series identity.
- Series extension behavior depends on classifying RT-step trends (mass-independent / increasing / decreasing with mass) of the first three members; ambiguous trends may constrain search windows incorrectly.
- Requires a pre-filtered, well-formatted peak table; features with mz ≤ 500 or rt ≤ 500 s may need exclusion as in the mtbls1358 example.
- Interactive curation via ptbPlotInteractive requires a shiny runtime.

## Evidence

- [methods] Untargeted detection in a search window: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100"
- [methods] Series database overlap handling: "overlapping_ids <- sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5)"
- [methods] Peak-level annotation from sdb: "dual_annotated <- annotateHomologues(peak_table, sdb, ppm_matching_tolerance = 1, ppm_step_tolerance = 10,"
- [abstract] Skill definition: "homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series as well as their evaluation and management using interactive plots and simple local"
- [methods] Mtbls1358 prefilter: "peak_table <- mtbls1358 %>% filter(., mz > 500, rt > 500)"
- [methods] Visual exploration: "Results of both runs are visualized using static ggplots, but can be explored in more detail using the ptbPlotInteractive function of homologueDiscoverer."
- [supplementary] Expected annotation counts: [evidence span withheld — anchor status: not_found]
- [results] Cross-tool comparison caveat: [evidence span withheld — anchor status: not_found]
