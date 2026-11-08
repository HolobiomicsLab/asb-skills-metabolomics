---
name: homologue-annotation-count-verification
description: Use when reproducing a published homologue-series annotation run on an
  LC-MS peak table (mz/rt/intensity columns) and a specific annotated-peak count must
  be matched as validation.
when_to_use_negative:
- Input is raw LC-MS vendor data or centroided spectra rather than a prebuilt peak
  table — detectHomologues operates on peak tables with mz/rt columns, so peak picking
  must happen first.
- The analytes of interest are not homologue series (no repetitive unit increment
  such as the 44.02628 Da ethylene-oxide step) — the increment-search premise does
  not apply.
- The goal is de novo series discovery/visualization only (e.g. ptbPlotInteractive
  exploration) rather than reproducing a validated annotation count against a manuscript
  figure.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: homologueDiscoverer
  role: R package providing detectHomologues (untargeted and targeted), sdbCheckContained,
    sdbCreate, sdbPush and annotateHomologues used to build and apply the series database
    for count verification
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: execution environment for the homologueDiscoverer pipeline scripts
- name: tidyverse
  role: used alongside homologueDiscoverer for loading and filtering the peak table
    (e.g. filter on mz and rt) before annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:cdc1da64f03ddbaa068846e9448167e58ea9edc3ea73e27802d98c09d622e91b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - tidyverse
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# homologue-annotation-count-verification

## Summary

Verifies homologue series annotation counts by running homologueDiscoverer's dual (untargeted + targeted) detectHomologues workflow and checking that the annotated peak table reproduces a published count, e.g. 115 of 276 PEG17 peaks and 140 of 251 PEG70 peaks in the da Silva et al. (2019) PEG spiked plasma evaluation.

## When to use

Use when reproducing a published homologue-series annotation run on an LC-MS peak table (mz/rt/intensity columns) and a specific annotated-peak count must be matched as validation.

## When NOT to use

- Input is raw LC-MS vendor data or centroided spectra rather than a prebuilt peak table — detectHomologues operates on peak tables with mz/rt columns, so peak picking must happen first.
- The analytes of interest are not homologue series (no repetitive unit increment such as the 44.02628 Da ethylene-oxide step) — the increment-search premise does not apply.
- The goal is de novo series discovery/visualization only (e.g. ptbPlotInteractive exploration) rather than reproducing a validated annotation count against a manuscript figure.

## Inputs

- LC-MS peak table (e.g. PEG17-spiked plasma: 276 peaks with peak_id, mz, rt, intensity columns; PEG70-spiked plasma: 251 peaks), as a data.frame/tibble loadable in R

## Outputs

- annotated_peaks.tsv — annotated peak table with a homologue_id column per peak
- series database (sdb) object built from sdbCreate/sdbPush of non-overlapping homologue series
- verification count: number of peaks with non-zero homologue_id (expected 115 of 276 for PEG17; 140 of 251 for PEG70)

## How to apply

Load the spiked-plasma peak table (e.g. the PEG17 table with 276 peaks carrying peak_id, mz, rt, intensity columns) and run detectHomologues in untargeted mode searching increments between mz_min = 10 and mz_max = 20 with rt_min = 1, rt_max = 100, ppm_tolerance = 5 and min_series_length = 5. Run a second targeted detectHomologues call for the ethylene-oxide increment mz_steps = c(44.02628) with the same rt window and tolerances. Check overlap between the two runs with sdbCheckContained (ppm_tolerance = 0.5, rt_tolerance = 0.5), push non-overlapping series into the series database via sdbCreate and sdbPush, then re-annotate the original table with annotateHomologues using ppm_matching_tolerance = 1, ppm_step_tolerance = 10, rt_tolerance = 1 and min_match_length = 4. Write the result to annotated_peaks.tsv and confirm the row count and the number of peaks with a non-zero homologue_id match the published figures (115/276 for PEG17; 140/251 for PEG70).

## Related tools

- **homologueDiscoverer** (R package providing detectHomologues (untargeted and targeted), sdbCheckContained, sdbCreate, sdbPush and annotateHomologues used to build and apply the series database for count verification) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (execution environment for the homologueDiscoverer pipeline scripts)
- **tidyverse** (used alongside homologueDiscoverer for loading and filtering the peak table (e.g. filter on mz and rt) before annotation)

## Examples

```
annotated <- annotateHomologues(peak_table, sdb, ppm_matching_tolerance = 1, ppm_step_tolerance = 10, rt_tolerance = 1, min_match_length = 4); write.table(annotated, "annotated_peaks.tsv")
```

## Evaluation signals

- The annotated table must contain exactly 276 rows for the PEG17 dataset (or 251 for PEG70), matching the input peak count.
- Exactly 115 of the 276 PEG17 peaks must carry a non-zero homologue_id assignment (140 of 251 for PEG70), matching the count reported in the manuscript.
- sdbCheckContained on the two detectHomologues runs must return overlapping series identifiers with ppm_tolerance = 0.5 and rt_tolerance = 0.5 before sdbPush of only non-overlapping series.
- annotateHomologues must be run with ppm_matching_tolerance = 1, ppm_step_tolerance = 10, rt_tolerance = 1 and min_match_length = 4; deviations in these parameters invalidate count comparability.
- The targeted run must use the ethylene-oxide increment mz_steps = c(44.02628) to reproduce the manuscript's dual-annotation strategy.

## Limitations

- The reference counts (115/276 PEG17, 140/251 PEG70) are tied to the da Silva et al. (2019) PEG spiked plasma datasets and the exact parameter settings; other datasets or tolerances will yield different counts.
- Counts depend on the combined untargeted (mz_min = 10 to mz_max = 20) and targeted (44.02628) detection runs plus database merging — running only one mode will not reproduce the published annotation.
- min_match_length = 4 during annotation differs from min_series_length = 5 during detection, so annotated counts can be lower than detected series memberships.

## Evidence

- [methods] dual-annotation via untargeted + targeted detectHomologues: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, 
                                            rt_min = 1, rt_max = 100,"
- [methods] targeted ethylene-oxide increment: "annotated_peak_table_02 <- detectHomologues(peak_table, mz_steps = c(44.02628),
                                            rt_min = 1, rt_max = 100,"
- [methods] series database overlap check and merge: "overlapping_ids <- sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5)"
- [methods] final annotation step with tolerances: "dual_annotated <- annotateHomologues(peak_table, sdb, 
                                     ppm_matching_tolerance = 1, 
                                     ppm_step_tolerance = 10,"
- [abstract] package purpose: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
