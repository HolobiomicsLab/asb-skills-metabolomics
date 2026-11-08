---
name: untargeted-mz-window-series-search
description: 'Use when an LC-MS peak table (mz, rt, intensity columns) must be searched
  for homologue series with an unknown m/z increment: no candidate step (e.g. 44.02628)
  is available, so a Da-range sweep (mz_min-mz_max) is required.'
when_to_use_negative:
- A specific m/z increment is already hypothesised (e.g. 44.02628 for repeated units)
  — run targeted detectHomologues with mz_steps instead of sweeping a window
- Input is raw spectra (vendor formats or mzML) rather than a processed peak table
  with mz, rt and intensity columns
- The goal is series-database management or re-annotation of an existing table (sdbCreate,
  sdbPush, annotateHomologues) rather than initial series discovery
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: homologueDiscoverer
  role: R package providing detectHomologues for untargeted m/z-window homologue series
    detection (initialize 2-tuples, expandChains), returning a homologue_id-annotated
    peak table
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Language runtime in which the homologueDiscoverer detectHomologues call is
    executed
- name: tidyverse
  role: Used to load and filter the peak table prior to detection (e.g. filter(.,
    mz > 500, rt > 500))
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:dd5cfb87b392e896d960cf1d311e3e990cbfda60ff465f5f66f071e384552f9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# untargeted-mz-window-series-search

## Summary

Untargeted detection of homologue series in LC-MS peak tables using homologueDiscoverer's detectHomologues in increment-window mode: 2-tuples are seeded from root peaks across an m/z increment range (e.g. 10-20 Da) and chains are extended under ppm and retention-time constraints, yielding a homologue_id-annotated peak table.

## When to use

Use when an LC-MS peak table (mz, rt, intensity columns) must be searched for homologue series with an unknown m/z increment: no candidate step (e.g. 44.02628) is available, so a Da-range sweep (mz_min-mz_max) is required.

## When NOT to use

- A specific m/z increment is already hypothesised (e.g. 44.02628 for repeated units) — run targeted detectHomologues with mz_steps instead of sweeping a window
- Input is raw spectra (vendor formats or mzML) rather than a processed peak table with mz, rt and intensity columns
- The goal is series-database management or re-annotation of an existing table (sdbCreate, sdbPush, annotateHomologues) rather than initial series discovery

## Inputs

- Pre-processed LC-MS peak table (data.frame/tibble with mz, rt, intensity columns; e.g. the mtbls1358 subset shipped in the homologueDiscoverer R package)
- Untargeted search settings: increment window mz_min-mz_max in Da, retention-time bounds rt_min-rt_max in seconds, ppm tolerance (e.g. 5), min_series_length (e.g. 5)

## Outputs

- Annotated peak table identical to the input plus a homologue_id column per peak (e.g. saved as annotated_peak_table_01.csv)
- Optional static or interactive series visualisations via plotAnnotatedStatic / ptbPlotInteractive

## How to apply

Load a pre-processed peak table with mz, rt and intensity columns — e.g. the mtbls1358 subset bundled with homologueDiscoverer — and filter it first to shrink the search space (article example: mz > 500, rt > 500 seconds). Call detectHomologues without mz_steps but with an untargeted increment window (mz_min = 10, mz_max = 20), retention-time bounds rt_min = 1 and rt_max = 100 seconds, ppm_tolerance = 5, and min_series_length = 5. Internally the two-phase routine (initializeChains/initializeUntargeted followed by expandChains) seeds a 2-tuple for each root peak using a candidate increment drawn from the window, checks for a second peak within the search window, and extends each chain only where consecutive m/z steps match the tuple's increment within the ppm tolerance and the relative rt tolerances. Series shorter than min_series_length (≥ 5 members in the example) are discarded; surviving series are annotated back onto the filtered table as a homologue_id column. The rationale for sweeping a window rather than fixing mz_steps (targeted mode) is to surface series whose increment was not hypothesised in advance; such overlapping results can later be reconciled with targeted runs via sdbCheckContained before pushing into a series database.

## Related tools

- **homologueDiscoverer** (R package providing detectHomologues for untargeted m/z-window homologue series detection (initialize 2-tuples, expandChains), returning a homologue_id-annotated peak table) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Language runtime in which the homologueDiscoverer detectHomologues call is executed)
- **tidyverse** (Used to load and filter the peak table prior to detection (e.g. filter(., mz > 500, rt > 500)))

## Examples

```
annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100)
```

## Evaluation signals

- Output CSV (annotated_peak_table_01.csv) exists and preserves the original mz, rt and intensity columns while adding a homologue_id annotation column
- Every reported series contains at least min_series_length members (≥ 5 in the article's run)
- Consecutive member m/z differences within each series fall inside the mz_min-mz_max window (10-20 Da) when compared at the 5 ppm tolerance
- Retention-time gaps between consecutive homologues lie within rt_min = 1 to rt_max = 100 seconds
- The untargeted run surfaces series with increments beyond any fixed targeted step (e.g. 44.02628), demonstrating the window sweep actually explored the range

## Limitations

- detectHomologues consumes a processed peak table (mz/rt/intensity), not raw spectra; peaks must be pre-filtered (article example: mz > 500, rt > 500 seconds on the mtbls1358 subset)
- Untargeted window sweeps produce many candidate series that can overlap targeted runs (e.g. increment 44.02628); the article's workflow reconciles this with sdbCheckContained overlap detection before sdbPush into the series database
- Chain survival depends on the chosen ppm tolerance and rt window during expandChains; series shorter than min_series_length are silently dropped, so overly strict thresholds can remove real homologues

## Evidence

- [abstract] untargeted detection of homologue series: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] untargeted detectHomologues call with increment window and rt bounds: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100,"
- [methods] peak table provenance and pre-filtering: "The peak table is loaded from the homologueDiscoverer package and immediately filtered."
- [methods] mtbls1358 example dataset: "We make use of a subset of the mtbls1358 dataset for this example run."
