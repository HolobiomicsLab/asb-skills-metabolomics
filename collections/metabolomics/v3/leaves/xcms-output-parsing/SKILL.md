---
name: xcms-output-parsing
description: Use when unaligned and aligned XCMS output tables (CSV) from LC-HRMS
  NPP must be loaded and matched to an mzRAPP benchmark (e.g. MTBLS267-derived Benchmark.csv)
  to quantify peak picking and alignment reliability.
when_to_use_negative:
- The NPP output comes from a tool other than XCMS (e.g. MZmine 2 or MS-DIAL) — select
  the corresponding tool instead, since the parsing differs per tool export format
- Only raw mzML files are available and no benchmark exists — benchmark generation
  must be done first before assessing XCMS output
- The goal is targeted quantification rather than reliability assessment of non-targeted
  pre-processing
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: mzRAPP
  role: Parses XCMS output tables and matches their peaks against the benchmark to
    compute reliability metrics
  repo: https://github.com/YasinEl/mzRAPP
- name: XCMS
  role: Source of the unaligned/aligned peak tables being parsed and assessed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:3cc465fb162c8a8d340412969b4ed229c59daf7a4cb577cb3a9d5d0dd3f6f83c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# xcms-output-parsing

## Summary

Parses XCMS peak-picking and alignment output tables (unaligned and aligned CSVs) so they can be matched against an mzRAPP-generated benchmark for non-targeted pre-processing reliability assessment.

## When to use

Use when unaligned and aligned XCMS output tables (CSV) from LC-HRMS NPP must be loaded and matched to an mzRAPP benchmark (e.g. MTBLS267-derived Benchmark.csv) to quantify peak picking and alignment reliability.

## When NOT to use

- The NPP output comes from a tool other than XCMS (e.g. MZmine 2 or MS-DIAL) — select the corresponding tool instead, since the parsing differs per tool export format
- Only raw mzML files are available and no benchmark exists — benchmark generation must be done first before assessing XCMS output
- The goal is targeted quantification rather than reliability assessment of non-targeted pre-processing

## Inputs

- Unaligned XCMS peak-picking output table (CSV, e.g. XCMS_unaligned_run3.csv)
- Aligned XCMS output table (CSV, e.g. XCMS_aligned_run3.csv)
- mzRAPP benchmark CSV (e.g. Benchmark.csv derived from MTBLS267 mzML files)
- Sample-group file and target file as used for benchmark generation

## Outputs

- Matched peak sets linking benchmark entries to XCMS found/split/missing peaks
- NPP performance metrics for peak picking, alignment, and post-alignment detection
- Exported metrics table (e.g. xcms_run3_assessment_metrics.csv)

## How to apply

Export the XCMS peak table in unaligned form and the aligned peak table as CSV files (in the mzRAPP example these are XCMS_unaligned_run3.csv and XCMS_aligned_run3.csv), then supply them together with the benchmark CSV in the 'Setup NPP assessment' tab with 'XCMS' selected as the non-targeted tool. mzRAPP matches benchmark peaks to the XCMS peak-picking and alignment outputs using its compare_peaks/match_peaks_to_benchmark pipeline. Inspect the three overview boxes in the 'View NPP assessment' tab — Post Alignment, Peak Picking, and Alignment — and record the metric ranges: about 93-99% of peaks detected post-alignment, 3-20% degenerated isotopologue ratios, and about 4-10% peaks lost during alignment. Because mzRAPP bootstraps confidence intervals, slight run-to-run variation is expected, so validation is by falling within these reported ranges rather than exact reproduction. Finally export the computed performance metrics (e.g. to xcms_run3_assessment_metrics.csv).

## Related tools

- **mzRAPP** (Parses XCMS output tables and matches their peaks against the benchmark to compute reliability metrics) — https://github.com/YasinEl/mzRAPP
- **XCMS** (Source of the unaligned/aligned peak tables being parsed and assessed)

## Examples

```
library(mzRAPP); callmzRAPP()
```

## Evaluation signals

- Post-alignment detection proportion falls in the reported 93-99% range
- Degenerated isotopologue ratio metric falls in the reported 3-20% range
- Peaks lost during alignment fall in the reported 4-10% range
- All three overview boxes (Post Alignment, Peak Picking, Alignment) populate without matching errors
- Exported metrics file (e.g. xcms_run3_assessment_metrics.csv) contains the computed performance metrics

## Limitations

- mzRAPP bootstraps confidence intervals, so computed metrics vary slightly between runs and exact-value reproduction is not expected
- Matching between benchmark and NPP output depends on the benchmark's isotopologue validation rules (e.g. Pearson correlation >= 0.85, area/height within 30% of predicted values), so benchmark quality bounds the assessment
- The assessment relies on the benchmark being generated from the same mzML files used for the XCMS run

## Evidence

- [readme] README overview of supported NPP tools: [evidence span withheld — anchor status: external_doc]
- [readme] README on benchmark-to-metrics pipeline: [evidence span withheld — anchor status: external_doc]
- [methods] Source card workflow on XCMS files and metric ranges: [evidence span withheld — anchor status: not_found]
- [results] Source card validation ranges: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract on mzRAPP benchmark use: "which generates and validates a benchmark from user-supplied information and later utilizes it for reliability assessment of data pre-processing"
