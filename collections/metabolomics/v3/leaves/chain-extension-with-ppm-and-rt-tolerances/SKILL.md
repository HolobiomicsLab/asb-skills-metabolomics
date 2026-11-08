---
name: chain-extension-with-ppm-and-rt-tolerances
description: Use when an LC-MS peak table (mz/rt/intensity) must be annotated with
  homologue series by seeding 2-tuples from root peaks and extending chains under
  an m/z increment, 5 ppm tolerance, and 1–100 s rt constraints (targeted mz_steps
  like 44.02628 or untargeted 10–20 Da).
when_to_use_negative:
- Input is raw LC-MS data (e.g. mzML) rather than an already peak-picked table with
  mz, rt, and intensity columns — chain extension operates on a filtered peak table.
- You need to evaluate or manage already-detected series (overlap checks, summaries,
  database push) rather than detect them — sdbCheckContained, sdbPush, and sdbSummarize
  cover that workflow.
- Expected m/z increments fall outside the untargeted search window (mz_min = 10 to
  mz_max = 20) and no targeted mz_steps are supplied.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: homologueDiscoverer
  role: Supplies detectHomologues with the initializeChains/initializeUntargeted/initializeTargeted
    and expandChains two-phase routine, the mtbls1358 example peak table, and the
    homologue_id annotation of the output
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Runtime language for executing the detectHomologues chain-extension routine
    and saving the annotated peak tables
- name: tidyverse
  role: Filters the loaded mtbls1358 peak table (mz > 500, rt > 500) before chain
    initialization and extension
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:36003ccc3789724ea063471b08a8f0dab56a06efb33a06545c0c725dcbaaba84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# chain-extension-with-ppm-and-rt-tolerances

## Summary

Reconstructs homologueDiscoverer's two-phase series detection: initializeTargeted/initializeUntargeted seed 2-tuples from root peaks, then expandChains extends each chain by screening the peak table for peaks matching the tuple's m/z increment within 5 ppm under the 1–100 s retention-time window, returning homologue_id-annotated peak tables.

## When to use

Use when an LC-MS peak table (mz/rt/intensity) must be annotated with homologue series by seeding 2-tuples from root peaks and extending chains under an m/z increment, 5 ppm tolerance, and 1–100 s rt constraints (targeted mz_steps like 44.02628 or untargeted 10–20 Da).

## When NOT to use

- Input is raw LC-MS data (e.g. mzML) rather than an already peak-picked table with mz, rt, and intensity columns — chain extension operates on a filtered peak table.
- You need to evaluate or manage already-detected series (overlap checks, summaries, database push) rather than detect them — sdbCheckContained, sdbPush, and sdbSummarize cover that workflow.
- Expected m/z increments fall outside the untargeted search window (mz_min = 10 to mz_max = 20) and no targeted mz_steps are supplied.

## Inputs

- mtbls1358 peak table (data.frame with mz, rt, intensity columns) loaded from the homologueDiscoverer package, pre-filtered to mz > 500 and rt > 500 seconds
- Mode parameters: untargeted mz_min = 10 / mz_max = 20 (Da) or targeted mz_steps = c(44.02628)
- Extension constraints: rt_min = 1, rt_max = 100 (seconds), ppm_tolerance = 5, min_series_length = 5

## Outputs

- Annotated peak table (annotated_peak_table_01 / annotated_peak_table_02) preserving the original mz, rt, and intensity columns plus a homologue_id annotation column per peak
- Saved CSV files annotated_peak_table_01.csv and annotated_peak_table_02.csv

## How to apply

Load the mtbls1358 peak table from the homologueDiscoverer package and filter with tidyverse to mz > 500 and rt > 500 seconds. Call detectHomologues in untargeted 'increment' step mode with mz_min = 10 and mz_max = 20, or in targeted mode with mz_steps = c(44.02628), keeping rt_min = 1, rt_max = 100, ppm_tolerance = 5, and min_series_length = 5. Internally, the routine first checks for a second peak within the search window of each root peak to initialize a 2-tuple (initializeChains / initializeUntargeted / initializeTargeted); expandChains then uses the tuple's m/z increment and relative retention-time constraints to screen the peak table for candidate peaks that extend the series. Chains shorter than min_series_length (5) are dropped, and surviving series are returned as an annotated peak table in which every peak carries a homologue_id. Judge correctness by confirming the mz, rt, and intensity columns are preserved, consecutive series members respect the 5 ppm tolerance and the 1–100 s rt window, and each series has at least 5 members.

## Related tools

- **homologueDiscoverer** (Supplies detectHomologues with the initializeChains/initializeUntargeted/initializeTargeted and expandChains two-phase routine, the mtbls1358 example peak table, and the homologue_id annotation of the output) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Runtime language for executing the detectHomologues chain-extension routine and saving the annotated peak tables)
- **tidyverse** (Filters the loaded mtbls1358 peak table (mz > 500, rt > 500) before chain initialization and extension)

## Examples

```
annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628), rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5)
```

## Evaluation signals

- Output CSVs (annotated_peak_table_01.csv, annotated_peak_table_02.csv) exist and preserve the original mz, rt, and intensity columns plus a homologue_id annotation column.
- Successive members of each homologue_id series differ by the specified increment within the 5 ppm tolerance (44.02628 Da for the targeted run).
- All consecutive series members satisfy the relative retention-time window of rt_min = 1 to rt_max = 100 seconds.
- Every reported series contains at least min_series_length = 5 peaks.
- The untargeted run over mz_min = 10 to mz_max = 20 yields annotated_peak_table_01 whose series are plottable with plotAnnotatedStatic (or explorable via ptbPlotInteractive).

## Limitations

- Untargeted detection is bounded by the supplied m/z increment window (10–20 Da in the example); increments outside this range require targeted mz_steps.
- Series shorter than min_series_length (5 in the example) are discarded, so valid shorter homologue pairs or triples are not returned.
- Extension quality depends on the peak table's mass accuracy and rt reproducibility; peaks violating the 5 ppm or 1–100 s rt constraints break chains.

## Evidence

- [abstract] Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628),
                                            rt_min = 1, rt_max = 100,: "annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628),
                                            rt_min = 1, rt_max = 100,"
- [methods] annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, 
                                            rt_min = 1, rt_max = 100,: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, 
                                            rt_min = 1, rt_max = 100,"
- [methods] The peak table is loaded from the homologueDiscoverer package and immediately filtered.: "The peak table is loaded from the homologueDiscoverer package and immediately filtered."
- [full_text] first checks for a second peak within the search window of a root peak to initialize 2-tuples: [evidence span withheld — anchor status: not_found]
- [full_text] uses the m/z increment and retention time constraints of each tuple to screen the peak table for candidates to extend the series: [evidence span withheld — anchor status: not_found]
