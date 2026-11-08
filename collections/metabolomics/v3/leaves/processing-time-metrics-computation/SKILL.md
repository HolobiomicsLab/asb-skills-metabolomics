---
name: processing-time-metrics-computation
description: Use when a task requires quantifying computational plus manual effort
  of a semi-automated LC-MS library-building workflow and comparing it against a manual
  per-compound baseline to derive a fold-reduction metric.
when_to_use_negative:
- Input is a raw mzML corpus needing actual processing (this skill only recomputes
  reported timing metrics, not the ~30 min mzML processing itself)
- The analysis goal is RT or m/z library construction rather than efficiency benchmarking
tools:
- name: metScribeR
  role: The semi-automated workflow whose reported processing-time figures (automated
    plus manual review) are being benchmarked against the manual MassHunter baseline
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Language used to build the data frame of benchmark figures and compute the
    fold-reduction and per-standard effort metrics
  repo: https://github.com/ncats/metScribeR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:761c6b1654f067d85201162c1426c042cebb079f0f992d80e1cad08659b44fd7
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# processing-time-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the metScribeR processing-time efficiency benchmark by loading reported figures (603 IROA standards, ~10 s combined effort per standard, ~31 min automated processing, ~72 min visual inspection, ~4.5 min/compound manual MassHunter baseline) into an R data frame and recomputing the derived fold-reduction in total processing time.

## When to use

Use when a task requires quantifying computational plus manual effort of a semi-automated LC-MS library-building workflow and comparing it against a manual per-compound baseline to derive a fold-reduction metric.

## When NOT to use

- Input is a raw mzML corpus needing actual processing (this skill only recomputes reported timing metrics, not the ~30 min mzML processing itself)
- The analysis goal is RT or m/z library construction rather than efficiency benchmarking

## Inputs

- Reported benchmark figures: 603 IROA standards, ~10 s per-standard metScribeR effort, ~31 min automated processing, ~72 min visual inspection, ~4.5 min per compound manual MassHunter baseline
- R environment with base arithmetic/data-frame handling

## Outputs

- timing_benchmark_summary.csv with one row per metric (per-standard effort, total automated time, total manual review time, total baseline time, fold reduction)

## How to apply

Load the benchmark figures from the article into an R data frame: 603 IROA compounds, ~10 s combined computational and manual effort per metScribeR standard, ~31 min total automated processing, ~72 min manual visual peak inspection, and a 4.5 min per compound manual MassHunter baseline. Compute the total manual baseline time as 603 × 4.5 min ≈ 2713.5 min, and the total metScribeR time as 31 + 72 ≈ 103 min. Derive the fold reduction as the manual baseline total divided by the metScribeR total. Write one row per metric (per-standard effort, total automated time, total manual review time, total baseline time, fold reduction) to timing_benchmark_summary.csv. The rationale is that automated mzML processing plus brief single-click peak review scales far better than manual MassHunter compound-by-compound extraction.

## Related tools

- **metScribeR** (The semi-automated workflow whose reported processing-time figures (automated plus manual review) are being benchmarked against the manual MassHunter baseline) — https://github.com/ncats/metScribeR
- **R** (Language used to build the data frame of benchmark figures and compute the fold-reduction and per-standard effort metrics) — https://github.com/ncats/metScribeR

## Examples

```
df <- data.frame(n=603, manual_per=4.5, met_auto=31, met_review=72); df$baseline <- df$n*df$manual_per; df$fold <- df$baseline/(df$met_auto+df$met_review); write.csv(df,'timing_benchmark_summary.csv')
```

## Evaluation signals

- Recomputed fold-reduction must fall between 25 and 27 (article reports 26-fold)
- Per-standard combined effort must round to approximately 10 s
- Total metScribeR time equals automated (31 min) plus manual inspection (72 min) ≈ 103 min
- Total manual baseline equals 603 compounds × 4.5 min ≈ 2713.5 min
- timing_benchmark_summary.csv contains one row for each of the five required metrics

## Limitations

- Fold-reduction estimate depends on reported per-compound MassHunter time (4.5 min) and may not generalize to other laboratories or instruments
- The visual inspection time (~72 min) is operator-dependent, so the derived per-standard ~10 s effort is approximate

## Evidence
