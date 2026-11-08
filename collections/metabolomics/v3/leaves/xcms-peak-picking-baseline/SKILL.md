---
name: xcms-peak-picking-baseline
description: Reference for benchmarking a chromatography–MS marker-identification
  method against XCMS, or when peak picking on 2D m/z vs retention-time maps is needed
  and conventional XCMS peak detection is an acceptable baseline.
when_to_use_negative:
- The input data has already been processed into a feature/marker table and no baseline
  peak-picking comparison is required
- The goal is to use NPFimg's peak-free marker identification rather than to benchmark
  against conventional XCMS peak picking
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Serves as the widely used baseline method for peak detection and marker identification
    on the same chromatography–MS datasets, providing signal-acquisition and marker-identification
    error rates against which alternatives are compared.
  repo: http://github.com/poomcj/NPFimg
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:c8777d8ee1fe02d3e6debd317e7dc127591a2783e56720692336577023d6c561
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# xcms-peak-picking-baseline

## Summary

Run XCMS peak detection and marker identification on chromatography–MS data as the baseline comparator for evaluating alternative marker-identification methods such as NPFimg, quantifying signal-acquisition and marker-identification error rates.

## When to use

Reference for benchmarking a chromatography–MS marker-identification method against XCMS, or when peak picking on 2D m/z vs retention-time maps is needed and conventional XCMS peak detection is an acceptable baseline.

## When NOT to use

- The input data has already been processed into a feature/marker table and no baseline peak-picking comparison is required
- The goal is to use NPFimg's peak-free marker identification rather than to benchmark against conventional XCMS peak picking

## Inputs

- Gas chromatography–mass spectrometry (GC–MS) chromatography–MS data (e.g. the aroma odor and human breath example datasets distributed with the NPFimg repository)
- Two-dimensional MS maps (m/z vs retention time) to be peak-picked

## Outputs

- Per-dataset signal-acquisition outcomes from XCMS peak detection
- Per-dataset marker-identification outcomes from XCMS
- Counts of erroneous signal-acquisition events and erroneous marker identifications
- Signal-acquisition and marker-identification error rates per method and dataset (erroneous events / total evaluated signals or markers)
- Comparison table (e.g. xcms_vs_npfimg_error_rates.csv) with one row per method and dataset

## How to apply

Run XCMS on the same GC–MS chromatography–mass spectrometry datasets that the alternative method (e.g. NPFimg) processes, performing its conventional peak-detection workflow over the two-dimensional m/z vs retention-time map. Record the signal-acquisition outcomes and marker-identification outcomes produced by XCMS for each dataset. Count erroneous signal-acquisition events and erroneous marker identifications and convert them to error rates (erroneous events divided by total evaluated signals/markers). Assemble a per-method, per-dataset comparison table — one row per method and dataset, with columns for signal-acquisition error rate and marker-identification error rate — writing it to a CSV (e.g. xcms_vs_npfimg_error_rates.csv) alongside the challenger method's rates for direct comparison.

## Related tools

- **XCMS** (Serves as the widely used baseline method for peak detection and marker identification on the same chromatography–MS datasets, providing signal-acquisition and marker-identification error rates against which alternatives are compared.) — http://github.com/poomcj/NPFimg

## Evaluation signals

- The comparison table exists and contains one row per method (NPFimg and XCMS) and per dataset (aroma odor and human breath GC–MS), with both error-rate metrics filled in.
- Error rates are computed as erroneous events divided by the total evaluated signals/markers for each method and dataset.
- Both signal-acquisition error rate and marker-identification error rate are reported for XCMS, so the rates can be directly compared with NPFimg's values.
- The CSV (e.g. xcms_vs_npfimg_error_rates.csv) opens cleanly and its schema matches the planned columns: method, dataset, signal-acquisition error rate, marker-identification error rate.

## Limitations

- Conventional peak picking suffers from false peak detections, which is the main reliability weakness this baseline skill inherits when evaluating marker-identification alternatives.
- The article demonstrates the benchmark on GC–MS data (aroma odor and human breath); applicability to other chromatography–MS modalities is implied but not demonstrated in the provided context.

## Evidence
