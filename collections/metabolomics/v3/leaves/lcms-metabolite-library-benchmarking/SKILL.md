---
name: lcms-metabolite-library-benchmarking
description: Use when quantifying the computational and manual effort of an LC-MS
  metabolite library construction workflow (e.g., metScribeR on mzML standard files)
  against a manual peak-picking baseline, to report per-standard effort and total
  time savings.
when_to_use_negative:
- Input is already a processed RT/m/z library CSV (exported_metScribeR_library.csv)
  — benchmarking raw-time comparison is not needed
- The goal is accuracy validation (e.g., RT correlation of 0.99) rather than effort/time
  measurement
- The dataset lacks a defined manual-processing baseline time per compound, making
  a fold-reduction comparison uncomputable
tools:
- name: metScribeR
  role: Source of the automated processing time (~31 min for 603 standards) and the
    Shiny app whose visual inspection time (~72 min) is measured in the benchmark
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Language used to load benchmark figures, compute totals and the derived fold
    reduction, and write timing_benchmark_summary.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:a478b4c9c33d00169a714379345e76816dda79cd4bd11c0b4a7afff9c2b584af
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - R
  - metScribeR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lcms-metabolite-library-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the metScribeR processing-time efficiency benchmark, comparing automated library construction plus visual peak review against a manual MassHunter-based baseline to derive a fold reduction in total processing time per metabolite standard.

## When to use

Use when quantifying the computational and manual effort of an LC-MS metabolite library construction workflow (e.g., metScribeR on mzML standard files) against a manual peak-picking baseline, to report per-standard effort and total time savings.

## When NOT to use

- Input is already a processed RT/m/z library CSV (exported_metScribeR_library.csv) — benchmarking raw-time comparison is not needed
- The goal is accuracy validation (e.g., RT correlation of 0.99) rather than effort/time measurement
- The dataset lacks a defined manual-processing baseline time per compound, making a fold-reduction comparison uncomputable

## Inputs

- Article-reported benchmark figures (603 IROA compounds, ~10 s/standard metScribeR effort, ~31 min automated processing, ~72 min visual inspection, 4.5 min/compound manual MassHunter baseline)
- metScribeR timing telemetry from the Shiny app processing run (mzML inputs for standards in pos/neg ESI modes)
- R session for computation and CSV export

## Outputs

- timing_benchmark_summary.csv with one row per metric: per-standard effort, total automated time, total manual review time, total baseline time, and fold reduction

## How to apply

Load the reported benchmark figures into an R data frame: 603 IROA compounds, ~10 s combined computational and manual effort per metScribeR standard, ~31 min total automated metScribeR processing, ~72 min manual visual peak inspection, and a 4.5 min per compound manual MassHunter baseline. Compute the manual baseline total as 603 × 4.5 ≈ 2713.5 min and the metScribeR total as 31 + 72 ≈ 103 min, with per-standard combined effort of ≈10 s. Derive the fold reduction as the manual baseline total divided by the metScribeR total, and write one row per metric (per-standard effort, total automated time, total manual review time, total baseline time, fold reduction) to timing_benchmark_summary.csv. Validate that the recomputed fold reduction falls between 25 and 27 (article reports 26-fold) and that per-standard effort rounds to approximately 10 s.

## Related tools

- **metScribeR** (Source of the automated processing time (~31 min for 603 standards) and the Shiny app whose visual inspection time (~72 min) is measured in the benchmark) — https://github.com/ncats/metScribeR
- **R** (Language used to load benchmark figures, compute totals and the derived fold reduction, and write timing_benchmark_summary.csv)

## Examples

```
R: manual_total <- 603*4.5; metScribeR_total <- 31+72; fold <- manual_total/metScribeR_total  # expect ~26
```

## Evaluation signals

- Recomputed fold-reduction value must fall between 25 and 27 (article reports 26-fold reduction)
- Per-standard combined effort must round to approximately 10 s
- Total metScribeR processing time (automated + manual inspection) must equal ~103 min (31 + 72)
- Total manual baseline time must equal ~2713.5 min (603 × 4.5)
- timing_benchmark_summary.csv must contain exactly one row per metric with no missing values

## Limitations

- Timing figures are article-reported estimates; actual runtimes depend on data size (README notes ~30 min for 12000 mzML files in the initial computation) and hardware
- Manual visual inspection time (~72 min) is user-dependent and may vary with reviewer experience and number of peaks passing filters
- Large submissions take time: the initial computation after clicking submit is the long step, and MoNA MS/MS search speed (if included) depends on server speed and can be slow

## Evidence
