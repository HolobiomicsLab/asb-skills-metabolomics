---
name: metabolomics-feature-annotation
description: Use when an LC-MS peak/feature table contains unannotated features suspected
  of forming homologue series (repeating mass differences with a shared RT trend)
  and you need series identifiers attached per feature.
when_to_use_negative:
- The goal is single-molecule metabolite identification against MS/MS spectra or databases
  (e.g. spectral library matching) rather than series-level homologue annotation
- Input is raw vendor/centroided data without a peak table — homologueDiscoverer operates
  on pre-picked feature tables, not mzML runs directly
- Features are already annotated with homologue_id from a prior run and only need
  visualization or database management (use plotAnnotatedStatic/ptbPlotInteractive
  or sdb tools instead of re-running detection)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: homologueDiscoverer
  role: Core R package providing detectHomologues (targeted/untargeted series detection),
    annotateHomologues, sdbCreate/sdbPush/sdbCheckContained/sdbSummarize, and plotting
    functions
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Runtime environment for the package and its vignette-driven workflow
- name: tidyverse
  role: dplyr::filter is used to subset the mtbls1358 peak table (mz > 500, rt > 500)
    before detection and to exclude overlapping series IDs before sdbPush
- name: ggplot2
  role: Backs plotAnnotatedStatic for static visualization of annotated peak tables
    grouped by homologue_id
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:4111502878f6d1692dacff68a6893c33db1f4b7a06da4e79a753f0c2847d5822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - ggplot2
  - homologueDiscoverer
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-feature-annotation

## Summary

Annotates LC-MS metabolomics features into homologue series by detecting recurring mass differences (e.g. 10–20 Da increments) paired with consistent retention-time behaviour, using the R package homologueDiscoverer in untargeted or targeted search modes.

## When to use

Use when an LC-MS peak/feature table contains unannotated features suspected of forming homologue series (repeating mass differences with a shared RT trend) and you need series identifiers attached per feature.

## When NOT to use

- The goal is single-molecule metabolite identification against MS/MS spectra or databases (e.g. spectral library matching) rather than series-level homologue annotation
- Input is raw vendor/centroided data without a peak table — homologueDiscoverer operates on pre-picked feature tables, not mzML runs directly
- Features are already annotated with homologue_id from a prior run and only need visualization or database management (use plotAnnotatedStatic/ptbPlotInteractive or sdb tools instead of re-running detection)

## Inputs

- LC-MS peak/feature table (data.frame/tibble) with at minimum mz, rt, and sample-intensity columns (e.g. the mtbls1358 subset bundled with homologueDiscoverer)
- Optional series database (sdb) from prior sdbCreate for re-annotation
- Optional target mass increment(s) for targeted mode, e.g. c(44.02628)

## Outputs

- Annotated peak table (e.g. annotated_peak_table_01) where each feature row carries a homologue_id assignment
- Series database (sdb) from sdbCreate with sample_origin metadata
- Summary object from sdbSummarize; overlapping-id vector from sdbCheckContained

## How to apply

Load a peak table (e.g. the mtbls1358 subset shipped with homologueDiscoverer) and pre-filter with dplyr::filter to a relevant region, e.g. mz > 500 and rt > 500 seconds, to reduce the search space. For untargeted detection, call detectHomologues with search_mode untargeted and step_mode = "increment", setting a mass-difference window (mz_min = 10 to mz_max = 20), an RT window (rt_min = 1 to rt_max = 100 seconds), ppm_tolerance = 5, and min_series_length = 5. The algorithm assesses the RT step trend of the first three series members — mass-independent, increasing with mass, or decreasing with mass — and uses that trend to constrain how the series is extended. For a known increment (e.g. 44.02628, a C2H4O repeat), run targeted mode with mz_steps. The output annotated peak table carries a homologue_id per feature; manage detected series with sdbCreate, sdbCheckContained (ppm_tolerance = 0.5, rt_tolerance = 0.5), and sdbPush, and attach database entries back to a raw table via annotateHomologues with ppm_matching_tolerance = 1 and ppm_step_tolerance = 10.

## Related tools

- **homologueDiscoverer** (Core R package providing detectHomologues (targeted/untargeted series detection), annotateHomologues, sdbCreate/sdbPush/sdbCheckContained/sdbSummarize, and plotting functions) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Runtime environment for the package and its vignette-driven workflow)
- **tidyverse** (dplyr::filter is used to subset the mtbls1358 peak table (mz > 500, rt > 500) before detection and to exclude overlapping series IDs before sdbPush)
- **ggplot2** (Backs plotAnnotatedStatic for static visualization of annotated peak tables grouped by homologue_id)

## Examples

```
annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, min_series_length = 5)
```

## Evaluation signals

- Returned annotated peak table must contain a homologue_id column with no NAs among members of detected series and each series having at least min_series_length = 5 members
- Within each homologue_id, consecutive member mass differences should fall inside the mz_min–mz_max window (10–20 Da) within the specified ppm_tolerance = 5
- RT step direction within each series must be consistent with the trend assessed from the first three members (mass-independent, increasing, or decreasing with mass)
- Re-annotating the original peak table with annotateHomologues (ppm_matching_tolerance = 1) should recover the same series assignments stored in the sdb
- sdbCheckContained on a second run (e.g. targeted 44.02628) should identify overlapping series IDs before pushing only non-overlapping ones with sdbPush

## Limitations

- Detection quality depends on pre-filtering the peak table (the vignette uses mz > 500, rt > 500 on mtbls1358); running on the full unfiltered table dilutes the search space
- Targeted mode requires a known mass increment (e.g. 44.02628) and will not discover unexpected series; untargeted mode is bounded by the mz_min–mz_max window searched
- Series extension is constrained by the RT trend inferred from only the first three members, so misordering or noisy RT early in a series can misdirect later search windows
- The GitHub repository ships no changelog, so version-specific behaviour changes are not documented

## Evidence

- [abstract] homologueDiscoverer enables targeted and untargeted detection of homologue series in metabolomics data, with interactive plots and local database functionalities for evaluation and management: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] Untargeted search window and RT/parameter setup used by detectHomologues in the vignette: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, 
                                            rt_min = 1, rt_max = 100,"
- [methods] Pre-filtering of the example mtbls1358 peak table prior to homologue detection: "peak_table <- mtbls1358 %>%
  filter(., mz > 500, rt > 500)"
- [methods] Re-annotation of a raw peak table against the series database with specific ppm tolerances: "dual_annotated <- annotateHomologues(peak_table, sdb, 
                                     ppm_matching_tolerance = 1, 
                                     ppm_step_tolerance = 10,"
- [discussion] RT-trend constraint applied during series extension in the untargeted algorithm: [evidence span withheld — anchor status: not_found]
