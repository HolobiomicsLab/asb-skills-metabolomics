---
name: isotopologue-ratio-quality-assessment
description: Use when assessing peak-abundance quality of XCMS or MZmine 2 pre-processing
  output against an mzRAPP benchmark — i.e. quantifying what fraction of matched peaks
  show degenerated isotopologue ratios.
when_to_use_negative:
- Input is raw mzML without an mzRAPP benchmark — IR bias checks require the predicted
  isotopologue ratios generated from the target file and enviPat
- The target molecules have only one detectable isotopologue per adduct — mzRAPP only
  accepts isotopologues where the most abundant plus at least one additional isotopologue
  are found
- The goal is peak-picking recall or alignment-error counting rather than peak-abundance
  quality — use found/not-found peak counts and assess_alignment()/count_alignment_errors()
  instead
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: Implements check_IR_biases() and derive_performance_metrics() to compute degenerated-IR
    proportions from matched benchmark/NPP peak pairs
  repo: https://github.com/YasinEl/mzRAPP
- name: enviPat
  role: Predicts isotopologue patterns and ratios used as reference during benchmark
    generation (get_mz_table)
- name: XCMS
  role: Source of NPP output (unaligned/aligned peak tables) whose isotopologue-ratio
    quality is assessed
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:df3044ec8dd2fc914b1bba1831c8990668b047234393ce9f28be2aead61c980c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MSnbase
  - MZmine 2
  - R
  - XCMS
  - enviPat
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# isotopologue-ratio-quality-assessment

## Summary

Computes mzRAPP's isotopologue-ratio (IR) degeneration metric, which judges peak-abundance quality of NPP output by measuring how far measured isotopologue ratios drift from the enviPat-predicted values in the benchmark.

## When to use

Use when assessing peak-abundance quality of XCMS or MZmine 2 pre-processing output against an mzRAPP benchmark — i.e. quantifying what fraction of matched peaks show degenerated isotopologue ratios.

## When NOT to use

- Input is raw mzML without an mzRAPP benchmark — IR bias checks require the predicted isotopologue ratios generated from the target file and enviPat
- The target molecules have only one detectable isotopologue per adduct — mzRAPP only accepts isotopologues where the most abundant plus at least one additional isotopologue are found
- The goal is peak-picking recall or alignment-error counting rather than peak-abundance quality — use found/not-found peak counts and assess_alignment()/count_alignment_errors() instead

## Inputs

- mzRAPP benchmark data.table (Benchmark.csv) with enviPat-predicted isotopologue ratios for target molecules (e.g. the 47-molecule MTBLS267 benchmark, 157 features, 2870 peaks)
- Matched NPP peak/feature tables (e.g. XCMS_unaligned_run1.csv, XCMS_aligned_run1.csv imported via import_ungrouped_xcms/import_grouped_xcms)
- Instrument/resolution setting (e.g. OrbitrapXL/Velos R60000@400) and lowest isotopologue threshold (0.05) from benchmark generation

## Outputs

- Degenerated-isotopologue-ratio proportion with bootstrapped confidence intervals (alpha = 0.95), reported per pre-processing run
- Per-stage performance metric table (e.g. xcms_run1_npp_assessment.csv / mzmine2_npp_assessment_metrics.csv) including the IR-degeneration column

## How to apply

After peaks have been matched to the benchmark (match_peaks_to_benchmark / match_features_to_benchmark), run check_IR_biases() to flag peaks whose measured isotopologue ratios diverge from the enviPat-predicted ratios recorded in the benchmark, then fold the degeneration fraction into derive_performance_metrics() with bootstrapped confidence intervals (alpha = 0.95). During benchmark generation, low-abundance peaks are already quality-filtered: isotopologues more than 30% off the predicted area/height or with Pearson correlation < 0.85 against the most abundant isotopologue are removed, and only isotopologues at or above the lowest relative-abundance threshold (0.05) with at least two detectable isotopologues per adduct are retained. The resulting degenerated-IR proportion is tool- and run-dependent — reported example ranges are 28-53% for XCMS run 1, 3-20% for XCMS run 3, and 1-9% for MZmine 2 run 1 — so judge results against the expected range for the specific run rather than a single universal cutoff.

## Related tools

- **mzRAPP** (Implements check_IR_biases() and derive_performance_metrics() to compute degenerated-IR proportions from matched benchmark/NPP peak pairs) — https://github.com/YasinEl/mzRAPP
- **enviPat** (Predicts isotopologue patterns and ratios used as reference during benchmark generation (get_mz_table))
- **XCMS** (Source of NPP output (unaligned/aligned peak tables) whose isotopologue-ratio quality is assessed)

## Examples

```
library(mzRAPP); ir_metrics <- derive_performance_metrics(benchmark = Benchmark.csv, unaligned = import_ungrouped_xcms("XCMS_unaligned_run1.csv"), aligned = import_grouped_xcms("XCMS_aligned_run1.csv", algo = 'XCMS')) # after match_peaks_to_benchmark() and check_IR_biases()
```

## Evaluation signals

- XCMS run 1 assessment should report degenerated isotopologue ratios in the 28-53% range; run 3 in the 3-20% range; MZmine 2 run 1 in the 1-9% range
- Benchmark peaks with area or height more than 30% off the predicted value, or Pearson correlation < 0.85 vs. the most abundant isotopologue, must have been removed before IR metrics are computed
- Only isotopologues at or above the 0.05 relative-abundance threshold, with at least two isotopologues detected, are eligible for the metric
- Confidence intervals derived by bootstrapping (alpha = 0.95) may shift values slightly between runs — small variation is expected, not an error
- Exported metric table (e.g. xcms_run1_npp_assessment.csv) contains the degenerated-IR column alongside post-alignment detection and alignment-error counts

## Limitations

- Bootstrap-derived confidence intervals introduce element of randomness; computed values vary slightly between executions
- Only isotopologues where the main adduct plus at least one additional isotopologue are detected contribute — low or singly-detected isotopologues are excluded
- IR quality depends on benchmark-generation filters (peak-shape correlation with the most abundant isotopologue, IR-bias cutoff), so benchmark construction errors propagate into the metric

## Evidence

- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [methods] Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue: "Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue"
- [abstract] mzRAPP produces several performance metrics for different steps of the pre-processing workflow: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] Lowest iso. to be considered [%]:</b> Lowest relative isotopologue abundance to be considered for each molecule (&gt;= 0.05).: [evidence span withheld — anchor status: not_found]
