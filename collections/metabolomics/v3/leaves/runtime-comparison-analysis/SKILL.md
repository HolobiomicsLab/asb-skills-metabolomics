---
name: runtime-comparison-analysis
description: Use when MetaXtract has processed one or more Thermo RAW files and you
  need to quantify per-file wall-clock runtime, compare it against a reference hardware
  baseline, and produce a per-file runtime comparison table.
when_to_use_negative:
- The RAW files have not yet been processed by MetaXtract — no runtime_summary_*.tsv
  exists, so there are no per-file runtimes to compare.
- The task requires throughput benchmarks on non-Thermo vendor formats (e.g. Bruker,
  mzML), since MetaXtract currently supports only Thermo Fisher RAW files.
- The comparison should be against a hardware profile other than the reference Windows
  11 i9-13900HX laptop, and no such baseline has been recorded locally.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3174
tools:
- name: pandas
  role: Used to parse the runtime_summary_*.tsv files and compute mean/min/max runtime
    statistics for each processed RAW file.
- name: MetaXtract
  role: Produces the runtime_summary_*.tsv per RAW file during metadata annotation;
    the skill benchmarks its per-file wall-clock runtime.
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:eb4543f8bb30582c5560de625047e3e6adea6298a2ac233f0dfa66fc6edb517b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Rappsilber-Laboratory/MetaXtract
  repo_url: https://github.com/Rappsilber-Laboratory/MetaXtract
  tools:
  - MetaXtract
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# runtime-comparison-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks per-file wall-clock runtime of MetaXtract metadata annotation on Thermo RAW files by parsing runtime_summary_*.tsv outputs with pandas, computing summary statistics (mean/min/max), and comparing each file's runtime against a reference average (~40 s/file on a Windows 11 i9-13900HX laptop).

## When to use

Use when MetaXtract has processed one or more Thermo RAW files and you need to quantify per-file wall-clock runtime, compare it against a reference hardware baseline, and produce a per-file runtime comparison table.

## When NOT to use

- The RAW files have not yet been processed by MetaXtract — no runtime_summary_*.tsv exists, so there are no per-file runtimes to compare.
- The task requires throughput benchmarks on non-Thermo vendor formats (e.g. Bruker, mzML), since MetaXtract currently supports only Thermo Fisher RAW files.
- The comparison should be against a hardware profile other than the reference Windows 11 i9-13900HX laptop, and no such baseline has been recorded locally.

## Inputs

- runtime_summary_*.tsv files written by MetaXtract for each processed RAW file
- Thermo Fisher RAW files downloaded from PRIDE via the Snakemake workflow

## Outputs

- runtime_comparison.tsv — one row per processed RAW file with per-file runtime (s), deviation from the ~40 s reference, and per-file status
- summary statistics (mean, min, max) of per-file runtimes in seconds

## How to apply

Parse the runtime_summary_*.tsv files that MetaXtract writes for each processed RAW file using pandas, extracting the per-file runtime in seconds for each of the downloaded PRIDE RAW files. Compute summary statistics — mean, minimum, and maximum — across the per-file runtimes. Compare each file's runtime against the reported reference average of ~40 s per file on the reference Windows 11 laptop (Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070). Tabulate per-file runtime, deviation from the 40 s reference, and per-file status into a summary table. Save the table as runtime_comparison.tsv, one row per processed RAW file, with each runtime value a positive number.

## Related tools

- **pandas** (Used to parse the runtime_summary_*.tsv files and compute mean/min/max runtime statistics for each processed RAW file.)
- **MetaXtract** (Produces the runtime_summary_*.tsv per RAW file during metadata annotation; the skill benchmarks its per-file wall-clock runtime.) — https://github.com/Rappsilber-Laboratory/MetaXtract

## Examples

```
python -c "import pandas as pd, glob; files=glob.glob('runtime_summary_*.tsv'); df=pd.concat(pd.read_csv(f,sep='\t') for f in files); stats=df['runtime_s'].agg(['mean','min','max']); df['dev_from_40s']=df['runtime_s']-40; df.to_csv('runtime_comparison.tsv',sep='\t',index=False)"
```

## Evaluation signals

- runtime_comparison.tsv exists and is well-formed TSV with one row per processed RAW file.
- Every runtime value in each row is a positive number.
- Mean/min/max summary statistics are computed across all per-file runtimes.
- Each row contains a deviation from the ~40 s reference and a per-file status field.
- Deviations from the ~40 s reference are consistent with the hardware profile (Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070).

## Limitations

- The ~40 s reference average is tied to a specific Windows 11 laptop (Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070); runtimes on other hardware will differ and must be re-baselined.
- Benchmark covers only the last 20 RAW files submitted to PRIDE in January 2025, so results may not generalise to other acquisition years or file sizes.
- No version number is stated for MetaXtract, and no changelog is provided, so runtime may vary across unreported software revisions.

## Evidence

- [other] Per-file runtime extraction from MetaXtract TSV output: [evidence span withheld — anchor status: not_found]
- [other] Reference hardware baseline of ~40 s per file: [evidence span withheld — anchor status: not_found]
