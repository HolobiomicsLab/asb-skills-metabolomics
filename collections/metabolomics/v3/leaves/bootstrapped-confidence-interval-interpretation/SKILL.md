---
name: bootstrapped-confidence-interval-interpretation
description: Use when mzRAPP's derive_performance_metrics has produced Peak Picking
  (or alignment) metrics with bootstrapped confidence intervals on a Benchmark.csv-derived
  dataset and you must judge whether values such as the split-peaks percentage are
  within the expected range.
when_to_use_negative:
- The benchmark has not yet been generated and validated from user-supplied target/SampleGroups_MTBLS267.csv/Target_File_MTBLS267.csv
  inputs — match and metric derivation require a valid Benchmark.csv.
- The input is raw centroided mzML data rather than an NPP output table (XCMS, MZmine
  2, MS-DIAL, etc.) — this skill interprets post-matching metric intervals, not raw
  spectra.
- The NPP tool output has no mzRAPP import function among the five supported pre-processing
  tools, so matching against the benchmark cannot be performed.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: R tool that matches the benchmark against NPP outputs and derives performance
    metrics with bootstrapped confidence intervals for interpretation
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Implementation environment in which mzRAPP and its derive_performance_metrics/matching
    functions are run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:75750c9c1cdfc2911f393f236534ca36dd94fb0efe347883c68daaeb769222e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bootstrapped-confidence-interval-interpretation

## Summary

Interprets the NPP performance metrics that mzRAPP reports for benchmark-matched non-targeted pre-processing runs, where metrics such as the split-peaks proportion are accompanied by bootstrapped confidence intervals. For the MTBLS267/XCMS run 2 example, the split-peak percentage from the bootstrapped CI must fall within the 1–3% range stated in the article.

## When to use

Use when mzRAPP's derive_performance_metrics has produced Peak Picking (or alignment) metrics with bootstrapped confidence intervals on a Benchmark.csv-derived dataset and you must judge whether values such as the split-peaks percentage are within the expected range.

## When NOT to use

- The benchmark has not yet been generated and validated from user-supplied target/SampleGroups_MTBLS267.csv/Target_File_MTBLS267.csv inputs — match and metric derivation require a valid Benchmark.csv.
- The input is raw centroided mzML data rather than an NPP output table (XCMS, MZmine 2, MS-DIAL, etc.) — this skill interprets post-matching metric intervals, not raw spectra.
- The NPP tool output has no mzRAPP import function among the five supported pre-processing tools, so matching against the benchmark cannot be performed.

## Inputs

- Benchmark.csv — mzRAPP benchmark generated/validated from MTBLS267 mzML files (downloadable from the ucloud deposit)
- XCMS unaligned NPP output (XCMS_unaligned_run2.csv) imported via import_ungrouped_xcms
- XCMS aligned NPP output (XCMS_aligned_run2.csv) imported via import_grouped_xcms
- generate_options output for XCMS

## Outputs

- Peak Picking box performance metrics with bootstrapped confidence intervals (e.g. proportion of matches classified as split peaks)
- xcms_run2_assessment.csv containing all computed metrics for validation against the article's expected ranges

## How to apply

After matching benchmark peaks/features to the NPP output with match_peaks_to_benchmark and match_features_to_benchmark (which perform the non-equivalent joins between Benchmark.csv and tables like XCMS_aligned_run2.csv), call derive_performance_metrics and read the Peak Picking box metrics. Each metric — e.g. the proportion of matches classified as split peaks — is reported as a point estimate with a bootstrapped confidence interval rather than a single number. Interpretation hinges on comparing the bootstrapped interval against expected ranges: for the XCMS run 2 assessment on MTBLS267 the article states the split-peaks percentage should lie in the 1–3% range, so a value outside that interval signals a peak-picking problem. Write all derived metrics (with their intervals) to xcms_run2_assessment.csv so the interval check is auditable.

## Related tools

- **mzRAPP** (R tool that matches the benchmark against NPP outputs and derives performance metrics with bootstrapped confidence intervals for interpretation) — https://github.com/YasinEl/mzRAPP
- **R** (Implementation environment in which mzRAPP and its derive_performance_metrics/matching functions are run)

## Examples

```
library(mzRAPP); metrics <- derive_performance_metrics(...) # Peak Picking box: write xcms_run2_assessment.csv, check split-peak % CI within 1-3%
```

## Evaluation signals

- The split-peaks percentage from the bootstrapped confidence interval falls within the 1–3% range stated in the article for the XCMS run 2 assessment.
- Both NPP inputs (unaligned and aligned XCMS run 2 outputs) pass check_nonTargeted_input before matching.
- All Peak Picking box metrics are written to xcms_run2_assessment.csv with their confidence intervals.
- match_peaks_to_benchmark and match_features_to_benchmark produce non-empty classified match sets (found/split/missing peaks) for every benchmark entry.
- Metric point estimates are consistent with the interval bounds (interval contains the estimate and has plausible bootstrap width).

## Limitations

- mzRAPP supports only five of the most commonly used pre-processing tools; outputs from unsupported NPP tools cannot be assessed via this metric pipeline.
- Benchmark quality gates apply upstream: isotopologue peaks more than 30% off the predicted area/height or with Pearson correlation < 0.85 to the most abundant isotopologue are removed, which constrains which metrics get intervals.
- No changelog is provided for the mzRAPP repository, so version drift in metric definitions may not be traceable.

## Evidence

- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] The resulting benchmark dataset is used to extract different performance metrics for NPP performed on the same mzML files.: [evidence span withheld — anchor status: external_doc]
- [readme] mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files.: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [methods] Download the already prepared benchmark "Benchmark.csv" if you do not want to do the benchmark generation part of the example: "Download the already prepared benchmark "Benchmark.csv" if you do not want to do the benchmark generation part of the example"
