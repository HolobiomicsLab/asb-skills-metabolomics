---
name: untargeted-mass-increment-window-search
description: Use when the input is an LC-MS peak table (feature list with m/z, rt,
  intensities) and candidate homologue mass increments are unknown, so a bounded scan
  of mass differences (e.g. mz_min = 10 to mz_max = 20) must reveal series automatically.
when_to_use_negative:
- A specific homologue mass increment is already known — use targeted mode with mz_steps
  (e.g. c(44.02628)) instead of an untargeted window scan
- Input is raw vendor/mzML spectra rather than a peak table — homologueDiscoverer
  operates on already-peak-picked feature tables (it loads and filters a peak table
  directly)
- The m/z range of interest is unfiltered and large — untargeted increment searching
  over unfiltered tables is combinatorially expensive; pre-filter (e.g. mz > 500,
  rt > 500) first
edam_operation: http://edamontology.org/operation_3695
tools:
- name: homologueDiscoverer
  role: Provides detectHomologues with untargeted search_mode and increment step_mode
    (initializeUntargeted -> expandChains) that scans mass-difference windows and
    chains series members
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: R
  role: Execution language for running detectHomologues and handling the annotated
    peak table
  repo: https://github.com/kevinmildau/homologueDiscoverer
- name: tidyverse
  role: dplyr::filter is used to pre-filter the peak table (mz > 500, rt > 500) before
    untargeted detection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac647
    title: Homologue series detection and management in LC-MS data with <i>homologueDiscoverer</i>
  source_content_hash: sha256:faa40c5988c0fcba593feca8bbde2913f045e267ac651a4fdb39ec1df8cf40bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/homologueDiscoverer
  tools:
  - R
  - homologueDiscoverer
  - tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# untargeted-mass-increment-window-search

## Summary

Runs homologueDiscoverer's untargeted series detection (initializeUntargeted -> expandChains) over a filtered LC-MS peak table, scanning a mass-difference window (e.g. m/z 10-20) in increment step_mode to discover unknown homologue series without pre-specifying increments. It outputs an annotated peak table whose homologue_id assignments grow by chaining members while respecting RT and ppm constraints.

## When to use

Use when the input is an LC-MS peak table (feature list with m/z, rt, intensities) and candidate homologue mass increments are unknown, so a bounded scan of mass differences (e.g. mz_min = 10 to mz_max = 20) must reveal series automatically.

## When NOT to use

- A specific homologue mass increment is already known — use targeted mode with mz_steps (e.g. c(44.02628)) instead of an untargeted window scan
- Input is raw vendor/mzML spectra rather than a peak table — homologueDiscoverer operates on already-peak-picked feature tables (it loads and filters a peak table directly)
- The m/z range of interest is unfiltered and large — untargeted increment searching over unfiltered tables is combinatorially expensive; pre-filter (e.g. mz > 500, rt > 500) first

## Inputs

- LC-MS peak/feature table (data frame with mz, rt, and sample intensity columns), e.g. the mtbls1358 subset included in the homologueDiscoverer package
- detectHomologues parameter set: mz_min, mz_max, rt_min, rt_max, ppm_tolerance, min_series_length, search_mode = 'untargeted', step_mode = 'increment'

## Outputs

- Annotated peak table (data frame) carrying homologue_id assignments for each detected series, e.g. annotated_peak_table_01
- Optional static ggplot visualization via plotAnnotatedStatic / interactive exploration via ptbPlotInteractive
- Series database entries via sdbCreate for downstream management (sdbPush, sdbCheckContained, sdbSummarize)

## How to apply

Load the peak table (e.g. the mtbls1358 subset shipped with homologueDiscoverer) and pre-filter to the m/z/RT region of interest with dplyr::filter — in the vignette, mz > 500 and rt > 500 s — because untargeted chaining is combinatorial and filtering prunes implausible pairs. Call detectHomologues with search_mode = 'untargeted' and step_mode = 'increment', sweeping mass differences between mz_min and mz_max, accepting only candidate steps whose RT difference falls within rt_min to rt_max seconds (vignette: 1-100 s) and whose mass error is within ppm_tolerance (vignette: 5 ppm), and keeping series with at least min_series_length members (vignette: 5). During series extension (expandChains), once a chain has three members the algorithm assesses the retention-time step trend of the first three members — mass-independent, or increasing/decreasing with mass — and uses that trend to constrain the search windows for subsequently added members, which is the key behavior distinguishing untargeted expansion. The function returns the input peak table augmented with homologue_id annotations; write it to CSV and optionally visualize with plotAnnotatedStatic or ptbPlotInteractive. Correctness is judged by checking that each assigned series is internally consistent: successive members differ by a mass step inside the scanned window, RT steps stay in-window, and series meet the minimum length.

## Related tools

- **homologueDiscoverer** (Provides detectHomologues with untargeted search_mode and increment step_mode (initializeUntargeted -> expandChains) that scans mass-difference windows and chains series members) — https://github.com/kevinmildau/homologueDiscoverer
- **R** (Execution language for running detectHomologues and handling the annotated peak table) — https://github.com/kevinmildau/homologueDiscoverer
- **tidyverse** (dplyr::filter is used to pre-filter the peak table (mz > 500, rt > 500) before untargeted detection)

## Examples

```
annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, rt_min = 1, rt_max = 100, ppm_tolerance = 5, search_mode = "untargeted", step_mode = "increment", min_series_length = 5)
```

## Evaluation signals

- Every output homologue_id group has length >= min_series_length (5 in the vignette run)
- Consecutive members within each series differ by a mass step within [mz_min, mz_max] = [10, 20] within ppm_tolerance = 5 ppm
- Successive RT differences within each series fall inside [rt_min, rt_max] = [1, 100] seconds
- No feature appears in more than one homologue_id series unless explicitly re-annotated downstream
- The annotated table retains all original rows/columns of the input peak table, adding homologue_id without dropping features

## Limitations

- Untargeted expansion is constrained after three members by the observed RT-step trend (mass-independent, or increasing/decreasing with mass); chains whose RT behavior deviates from this trend after the first three members will not be extended accordingly
- Only mass differences within the user-specified [mz_min, mz_max] window are found; increments outside the window are invisible to the untargeted scan
- Performance depends on pre-filtering; the vignette demo filters to mz > 500 and rt > 500 s before searching

## Evidence

- [methods] untargeted mass-increment window search via detectHomologues: "annotated_peak_table_01 <- detectHomologues(peak_table, mz_min = 10, mz_max = 20, "
- [abstract] untargeted detection scope of the package: "Here, we present homologueDiscoverer, an R package that allows for the targeted and untargeted detection of homologue series"
- [methods] input dataset for the untargeted run: "We make use of a subset of the mtbls1358 dataset for this example run."
- [other] RT-step trend constraint during series extension (expandChains): [evidence span withheld — anchor status: not_found]
