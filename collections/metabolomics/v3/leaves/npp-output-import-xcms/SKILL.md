---
name: npp-output-import-xcms
description: Use when you have XCMS peak-picking/alignment output tables (e.g. XCMS_unaligned_run2.csv
  and XCMS_aligned_run2.csv) that must be loaded into mzRAPP for benchmark-based performance
  assessment.
when_to_use_negative:
- When the input is already a benchmark dataset or raw mzML data — use mzRAPP's benchmark-generation
  workflow (Generate Benchmark tab) instead of NPP-output import
- When the NPP output comes from a tool other than the five supported pre-processing
  tools (e.g. an unsupported custom pipeline) without an appropriate mzRAPP import
  function
- When only peak-annotation quality (not peak picking/alignment performance) is of
  interest, since this skill targets matching NPP peaks/features against the benchmark
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: R package providing import_ungrouped_xcms, import_grouped_xcms, check_nonTargeted_input,
    and benchmark-matching functions used to ingest and validate XCMS NPP outputs
  repo: https://github.com/YasinEl/mzRAPP
- name: XCMS
  role: Source non-targeted pre-processing tool whose unaligned and aligned output
    tables are imported and assessed
- name: R
  role: Runtime environment in which mzRAPP is implemented and the import/matching
    workflow is executed
  repo: https://github.com/YasinEl/mzRAPP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:257d9a1d42a22a8e7cdc87d0c92e65ca72c2628b026b1c4d721ee8c65cd4cec9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# npp-output-import-xcms

## Summary

Import and validate non-targeted pre-processing (NPP) output tables from XCMS — both unaligned (pre-grouping) and aligned (post-grouping) peak/feature tables — into mzRAPP so they can be joined against a benchmark for reliability assessment.

## When to use

Use when you have XCMS peak-picking/alignment output tables (e.g. XCMS_unaligned_run2.csv and XCMS_aligned_run2.csv) that must be loaded into mzRAPP for benchmark-based performance assessment.

## When NOT to use

- When the input is already a benchmark dataset or raw mzML data — use mzRAPP's benchmark-generation workflow (Generate Benchmark tab) instead of NPP-output import
- When the NPP output comes from a tool other than the five supported pre-processing tools (e.g. an unsupported custom pipeline) without an appropriate mzRAPP import function
- When only peak-annotation quality (not peak picking/alignment performance) is of interest, since this skill targets matching NPP peaks/features against the benchmark

## Inputs

- XCMS unaligned (pre-grouping) output table, e.g. XCMS_unaligned_run2.csv
- XCMS aligned (post-grouping) output table, e.g. XCMS_aligned_run2.csv
- Prepared benchmark dataset Benchmark.csv from the mzRAPP ucloud deposit
- Centroided mzML files from MTBLS267 (optional context for the underlying dataset)

## Outputs

- Validated mzRAPP-format ungrouped XCMS peak table
- Validated mzRAPP-format grouped (aligned) XCMS feature table
- Benchmark-to-NPP peak and feature match tables from match_peaks_to_benchmark and match_features_to_benchmark
- Performance metrics table, e.g. xcms_run2_assessment.csv, containing Peak Picking box metrics including split-peak proportion

## How to apply

Load the mzRAPP package in R and read the prepared Benchmark.csv downloaded from the ucloud deposit. Import the unaligned XCMS output with import_ungrouped_xcms (paired with generate_options for XCMS) and the aligned XCMS output with import_grouped_xcms, using the files supplied in the XCMS/MZmine ucloud deposit. Validate both NPP inputs with check_nonTargeted_input to confirm the tables conform to mzRAPP's expected schema before any matching. The rationale is that mzRAPP's compare/match functions perform non-equivalent joins between benchmark peaks and NPP peaks (match_peaks_to_benchmark) and between benchmark features and NPP features (match_features_to_benchmark), so correctly formatted imports are a prerequisite for computing metrics such as split peaks. After validation, derive performance metrics with derive_performance_metrics and write them out, e.g. to xcms_run2_assessment.csv.

## Related tools

- **mzRAPP** (R package providing import_ungrouped_xcms, import_grouped_xcms, check_nonTargeted_input, and benchmark-matching functions used to ingest and validate XCMS NPP outputs) — https://github.com/YasinEl/mzRAPP
- **XCMS** (Source non-targeted pre-processing tool whose unaligned and aligned output tables are imported and assessed)
- **R** (Runtime environment in which mzRAPP is implemented and the import/matching workflow is executed) — https://github.com/YasinEl/mzRAPP

## Examples

```
library(mzRAPP); ung <- import_ungrouped_xcms("XCMS_unaligned_run2.csv"); grp <- import_grouped_xcms("XCMS_aligned_run2.csv"); check_nonTargeted_input(ung); check_nonTargeted_input(grp)
```

## Evaluation signals

- check_nonTargeted_input passes without schema errors for both the ungrouped and grouped XCMS tables
- match_peaks_to_benchmark and match_features_to_benchmark produce non-empty match tables from the imported outputs
- The derived split-peaks proportion from bootstrapped confidence intervals falls within the 1-3% range stated in the article
- All computed Peak Picking box metrics are written to xcms_run2_assessment.csv

## Limitations

- Only the five most commonly used pre-processing tools are supported by mzRAPP; unsupported tool outputs require custom import handling
- Main-adduct-style constraints do not apply here, but benchmark validity still depends on the target file (e.g. SumForm_c without zero trailing element counts like C12H8N0S2) — mis-formatted upstream files propagate into matching
- Installation may require skipping package updates ('None' option) because of a known problem with the mzR package

## Evidence

- [other] mzRAPP's compare_peaks function performs non-equivalent joins between the benchmark and the supplied NPP output tables (e.g., the XCMS run 2 outputs): [evidence span withheld — anchor status: not_found]
- [abstract] mzRAPP supporting five of the most commonly used pre-processing tools: "supporting five of the most commonly used pre-processing tools"
- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] README use examples via vignette: [evidence span withheld — anchor status: external_doc]
