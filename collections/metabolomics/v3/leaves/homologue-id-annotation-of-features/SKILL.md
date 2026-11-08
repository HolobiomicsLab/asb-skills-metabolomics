---
name: homologue-id-annotation-of-features
description: Use when a metabolomics peak table with mz/rt/intensity columns needs
  homologue series grouping — e.g. recurring methylene-repeat or step-mass families
  — and each feature should receive a homologue_id annotation.
when_to_use_negative:
- Input is raw vendor or mzML data rather than an already-generated peak/feature table
  with mz and rt columns
- Series of interest are not related by consistent mass increments within an RT window
  (homologue series logic does not apply)
- A single targeted increment is already known and a full untargeted sweep over mz_min–mz_max
  windows would add noise
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: homologueDiscoverer
  role: R package providing detectHomologues, annotateHomologues, and the sdb* series-database
    functions used to detect, manage, and propagate homologue_id annotations
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: tidyverse
  role: Used to load and filter the peak table (e.g. filter(mz > 500, rt > 500)) before
    detection
- name: ggplot2
  role: Backing static visualization of annotated peak tables via plotAnnotatedStatic
- name: RColorBrewer
  role: Supplies the Set1 palette ramp for coloring series groupings in the annotated
    plots
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:2cbf1b0235b0354c3497285d37cc02c164e4e50748ac9eefe75cda751ee76f9d
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# homologue-id-annotation-of-features

## Summary

Annotates LC-MS peak table features with homologue_id assignments by detecting homologue series (peaks related by incremental mass steps within RT windows) using the homologueDiscoverer R package. Enables grouping of recurring, family-like metabolite signals for targeted and untargeted metabolomics data.

## When to use

Use when a metabolomics peak table with mz/rt/intensity columns needs homologue series grouping — e.g. recurring methylene-repeat or step-mass families — and each feature should receive a homologue_id annotation.

## When NOT to use

- Input is raw vendor or mzML data rather than an already-generated peak/feature table with mz and rt columns
- Series of interest are not related by consistent mass increments within an RT window (homologue series logic does not apply)
- A single targeted increment is already known and a full untargeted sweep over mz_min–mz_max windows would add noise

## Inputs

- LC-MS peak table with columns peak_id, mz, rt, intensity (e.g. the mtbls1358 peak table bundled with homologueDiscoverer)
- Optional targeted mz_steps list (e.g. c(44.02628)) for known homologue increments

## Outputs

- Annotated peak table with homologue_id assignments per peak
- Series database (sdb) of detected homologue series via sdbCreate
- sdbSummarize summary of series counts and memberships

## How to apply

Load the peak table (e.g. the mtbls1358 peak table bundled with the package) and filter to a defined mass/retention region — the article uses mz > 500 and rt > 500 s — to reduce spurious detections. Run detectHomologues in untargeted search_mode over a mass-step window such as mz 10–20 Da and rt 1–100 s, with ppm_tolerance = 5, min_series_length = 5 and step_mode = 'increment', so related peaks are assigned a shared homologue_id; a targeted run can instead pass explicit mz_steps such as c(44.02628) for known increments. Manage the results with the series database functions (sdbCreate, sdbCheckContained with ppm_tolerance = 0.5 / rt_tolerance = 0.5, sdbPush) to merge targeted and untargeted detections, then propagate annotations back onto the original table with annotateHomologues using ppm_matching_tolerance = 1 and ppm_step_tolerance = 10. Inspect results with plotAnnotatedStatic or the ptbPlotInteractive function and summarize via sdbSummarize.

## Related tools

- **homologueDiscoverer** (R package providing detectHomologues, annotateHomologues, and the sdb* series-database functions used to detect, manage, and propagate homologue_id annotations) — https://github.com/kevinmildau/homologueDiscoverer
- **tidyverse** (Used to load and filter the peak table (e.g. filter(mz > 500, rt > 500)) before detection)
- **ggplot2** (Backing static visualization of annotated peak tables via plotAnnotatedStatic)
- **RColorBrewer** (Supplies the Set1 palette ramp for coloring series groupings in the annotated plots)

## Examples

```
peak_table <- mtbls1358 %>% filter(., mz > 500, rt > 500); annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5, step_mode = 'increment')
```

## Evaluation signals

- Every peak with a non-NA homologue_id falls in a series of at least min_series_length = 5 members
- The untargeted run on the mtbls1358 subset should reproduce the reported scale: 275 homologue series groupings covering 1372 peaks (narrow search window, mz 10–20 Da / rt 1–100 s, ppm_tolerance = 5)
- sdbCheckContained returns the overlapping homologue_ids shared between untargeted and targeted (44.02628) runs, and sdbPush adds only non-overlapping series
- annotateHomologues with ppm_matching_tolerance = 1 matches series back onto the original peak table without duplicating assignments
- plotAnnotatedStatic/ptbPlotInteractive show consistent homologue_id groupings when grouped in the ggplot aes(group = homologue_id)

## Limitations

- A narrow search window yields substantial but incomplete grouping — 275 series / 1372 peaks on mtbls1358 is one operating point, not a ground truth
- Untargeted and targeted runs can detect overlapping series, requiring sdbCheckContained/sdbPush reconciliation before annotation
- Detection quality depends on the prior filtering of the peak table (mz > 500, rt > 500 in the tutorial), so results change with different cutoffs

## Evidence

- [abstract] untargeted detection of homologue series: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] peak table filter: "peak_table <- mtbls1358 %>
  filter(., mz > 500, rt > 500)"
- [methods] detectHomologues call: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100,"
- [methods] series database creation: "sdb <- sdbCreate(annotated_peak_table_01, sample_origin = "mtbls1358")"
- [methods] overlap check: "overlapping_ids <- sdbCheckContained(sdb, annotated_peak_table_02, ppm_tolerance = 0.5, rt_tolerance = 0.5)"
- [results] finding on series counts: [evidence span withheld — anchor status: not_found]
- [methods] annotation propagation: "dual_annotated <- annotateHomologues(peak_table, sdb, ppm_matching_tolerance = 1, ppm_step_tolerance = 10,"
