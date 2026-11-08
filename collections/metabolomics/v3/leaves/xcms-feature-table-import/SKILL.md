---
name: xcms-feature-table-import
description: Use when you have XCMS run output tables (unaligned and aligned CSVs,
  e.g. XCMS_unaligned_run1.csv / XCMS_aligned_run1.csv) and a Benchmark.csv that must
  be loaded into mzRAPP for NPP reliability assessment.
when_to_use_negative:
- Your NPP output comes from MZmine 2, MS-DIAL or other tools — mzRAPP requires their
  dedicated import functions, not the XCMS importers.
- You only have raw mzML files and no XCMS run output — this skill does not cover
  pre-processing or benchmark generation, only importing existing XCMS tables.
- The output tables are already post-alignment feature summaries only, and you need
  per-sample peak-level metrics — the unaligned (ungrouped) table is additionally
  required.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: Provides check_nonTargeted_input() and import_ungrouped_xcms()/import_grouped_xcms()
    (algo='XCMS') to validate and load the XCMS tables
  repo: https://github.com/YasinEl/mzRAPP
- name: XCMS
  role: Source of the unaligned/aligned output CSV tables being imported
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:be99a52c9a00fe5081da8f02121f4c68fbcb34035cec3e4ff080373365f85831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - XCMS
  - mzRAPP
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# xcms-feature-table-import

## Summary

Validates and imports unaligned and aligned XCMS peak/feature tables into mzRAPP's reliability-assessment workflow, converting standard XCMS CSV output into the benchmark-matching format via import_ungrouped_xcms()/import_grouped_xcms() with algo='XCMS'.

## When to use

Use when you have XCMS run output tables (unaligned and aligned CSVs, e.g. XCMS_unaligned_run1.csv / XCMS_aligned_run1.csv) and a Benchmark.csv that must be loaded into mzRAPP for NPP reliability assessment.

## When NOT to use

- Your NPP output comes from MZmine 2, MS-DIAL or other tools — mzRAPP requires their dedicated import functions, not the XCMS importers.
- You only have raw mzML files and no XCMS run output — this skill does not cover pre-processing or benchmark generation, only importing existing XCMS tables.
- The output tables are already post-alignment feature summaries only, and you need per-sample peak-level metrics — the unaligned (ungrouped) table is additionally required.

## Inputs

- Benchmark.csv (mzRAPP benchmark dataset)
- XCMS unaligned peak table (e.g. XCMS_unaligned_run1.csv)
- XCMS aligned/grouped feature table (e.g. XCMS_aligned_run1.csv)

## Outputs

- mzRAPP internal ungrouped peak table object
- mzRAPP internal grouped feature table object
- Validation report from check_nonTargeted_input()

## How to apply

Launch mzRAPP in R with callmzRAPP() (or script it) and supply the pre-generated benchmark (Benchmark.csv) together with the XCMS unaligned and aligned output tables exported from the XCMS run. First validate the inputs with check_nonTargeted_input() to catch format problems before matching. Then import the unaligned (peak-level) table with import_ungrouped_xcms(algo='XCMS') and the aligned/grouped (feature-level) table with import_grouped_xcms(algo='XCMS'), which normalizes the XCMS columns into mzRAPP's internal peak/feature representation. Correct imports are the prerequisite for match_peaks_to_benchmark(), match_features_to_benchmark() and downstream alignment-error and isotopologue-ratio metrics.

## Related tools

- **mzRAPP** (Provides check_nonTargeted_input() and import_ungrouped_xcms()/import_grouped_xcms() (algo='XCMS') to validate and load the XCMS tables) — https://github.com/YasinEl/mzRAPP
- **XCMS** (Source of the unaligned/aligned output CSV tables being imported)

## Examples

```
library(mzRAPP); callmzRAPP()  # then: check_nonTargeted_input(...); import_ungrouped_xcms(algo='XCMS'); import_grouped_xcms(algo='XCMS')
```

## Evaluation signals

- check_nonTargeted_input() reports no format errors for Benchmark.csv and both XCMS tables
- Imported benchmark-vs-NPP peak matching recovers the reference run-1 counts (2660/2870 benchmark peaks found; up to 12% split peaks), values allowed to vary slightly due to bootstrap randomness
- Downstream metrics stay in the reported ranges: 83-94% post-alignment detection, 28-53% degenerated isotopologue ratios, 19-35% peaks lost during alignment
- Both the unaligned AND aligned tables import successfully — alignment-error counting (assess_alignment()/count_alignment_errors()) is impossible without both

## Limitations

- mzRAPP requires centroided mzML-derived benchmarks; importing tables produced on profile-mode or unmatched mzML file sets will break peak matching
- Bootstrap-derived confidence intervals (alpha = 0.95) mean metric values vary slightly between runs
- The XCMS exporter choice matters — mzRAPP expects the specific unaligned/aligned CSV exports described in its documentation

## Evidence

- [methods] Validate the inputs with check_nonTargeted_input() and import the unaligned and grouped XCMS tables via import_ungrouped_xcms()/import_grouped_xcms() (algo='XCMS').: [evidence span withheld — anchor status: not_found]
- [methods] launch mzRAPP with callmzRAPP() and load the benchmark (Benchmark.csv), the unaligned XCMS output (XCMS_unaligned_run1.csv) and the aligned XCMS output: [evidence span withheld — anchor status: not_found]
- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [results] matching the BM against supplied NPP output tables (e.g., XCMS unaligned/aligned results), mzRAPP derives performance metrics covering missed peaks: [evidence span withheld — anchor status: not_found]
