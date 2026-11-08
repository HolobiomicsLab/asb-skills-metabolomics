---
name: cross-method-benchmark-comparison
description: Use when a new method (e.g. NPFimg) claims lower error rates of signal
  acquisition or marker identification than a baseline (e.g. XCMS) and the same raw
  chromatography–MS data can be processed by both.
when_to_use_negative:
- The input is already a processed feature/marker table — benchmarking requires running
  both methods end-to-end on the same raw MS maps so peak-picking behavior differs
  only by method.
- Only one method can be executed on the data (e.g. XCMS cannot be applied to the
  data format, or the image-based method lacks a suitable input representation) —
  a comparison needs both arms on identical inputs.
- The evaluation goal is runtime/memory benchmarking rather than error rates of signal
  acquisition and marker identification — the paper's claim is about correctness (lower
  error rates), not speed.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NPFimg
  role: 'Treatment arm: identifies multivariate chemo-/biomarker features from two-dimensional
    MS maps without conventional peak picking; its signal-acquisition and marker-identification
    error rates are the quantities being benchmarked'
  repo: https://github.com/poomcj/NPFimg
- name: XCMS
  role: 'Baseline arm: widely used peak detection and marker identification pipeline;
    its error rates on the same GC–MS data provide the comparison point'
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:02dd0b7d00f9819944006b37c2e7448e778f3a0e1e9c76d0acc3f8fff296cb7b
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/poomcj/NPFimg
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cross-method-benchmark-comparison

## Summary

Benchmark a newly proposed chromatography–MS data-processing method against an established baseline (here, NPFimg vs XCMS) by running both on the same GC–MS datasets and quantifying error rates of signal acquisition and marker identification. This skill converts a paper's comparative claim into a reproducible head-to-head comparison table.

## When to use

Use when a new method (e.g. NPFimg) claims lower error rates of signal acquisition or marker identification than a baseline (e.g. XCMS) and the same raw chromatography–MS data can be processed by both.

## When NOT to use

- The input is already a processed feature/marker table — benchmarking requires running both methods end-to-end on the same raw MS maps so peak-picking behavior differs only by method.
- Only one method can be executed on the data (e.g. XCMS cannot be applied to the data format, or the image-based method lacks a suitable input representation) — a comparison needs both arms on identical inputs.
- The evaluation goal is runtime/memory benchmarking rather than error rates of signal acquisition and marker identification — the paper's claim is about correctness (lower error rates), not speed.

## Inputs

- Raw GC–MS chromatography–MS datasets processed identically by both methods (e.g. aroma odor and human breath datasets from the NPFimg repository)
- Two-dimensional MS map (m/z vs retention time) representations of the data for the image-based method (NPFimg)
- Reference assignments for judging erroneous signal-acquisition events and erroneous marker identifications

## Outputs

- Comparison table (e.g. xcms_vs_npfimg_error_rates.csv) with one row per method × dataset, columns for signal-acquisition error rate and marker-identification error rate
- Per-method error counts (erroneous events / total evaluated signals and markers) for each dataset

## How to apply

Obtain the same raw GC–MS chromatography–MS datasets — in the NPFimg paper, the aroma odor and human breath case-study datasets distributed with the NPFimg repository (github.com/poomcj/NPFimg) — so both methods consume identical inputs. Run each method's native marker-feature identification on each dataset: NPFimg processes the two-dimensional MS map (m/z vs retention time) with image processing plus machine learning and skips conventional peak picking, while XCMS performs conventional peak detection. For each method × dataset combination, count erroneous signal-acquisition events and erroneous marker identifications, then compute error rates as erroneous events divided by the total evaluated signals/markers. Assemble a comparison table with one row per method and dataset, reporting both the signal-acquisition error rate and the marker-identification error rate for NPFimg versus XCMS. The expected outcome, per the paper's abstract, is that NPFimg shows lower error rates in both metrics than the widely used XCMS, which 'suffers from false peak detections'. Judge correctness by verifying the table contains both methods, both datasets, and both error-rate metrics, and that the claimed direction (NPFimg < XCMS) holds in the computed values.

## Related tools

- **NPFimg** (Treatment arm: identifies multivariate chemo-/biomarker features from two-dimensional MS maps without conventional peak picking; its signal-acquisition and marker-identification error rates are the quantities being benchmarked) — https://github.com/poomcj/NPFimg
- **XCMS** (Baseline arm: widely used peak detection and marker identification pipeline; its error rates on the same GC–MS data provide the comparison point)

## Examples

```
python npfimg_xcms_benchmark.py --datasets aroma,breath --methods npfimg --methods xcms --out xcms_vs_npfimg_error_rates.csv
```

## Evaluation signals

- The comparison table contains exactly the two methods (NPFimg, XCMS) as rows or columns and both datasets (aroma odor, human breath) as the other axis.
- Both error-rate metrics — signal-acquisition error rate and marker-identification error rate — are present for every method × dataset cell.
- Computed error rates are defined as erroneous events divided by total evaluated signals/markers, so values lie in [0, 1]; flag any negative or >1 entries as definition violations.
- Direction check: NPFimg's error rates should be lower than XCMS's for both datasets and both metrics, consistent with the paper's reported result.
- Both methods were run on the identical raw GC–MS input datasets (same files, same case studies) — confirm input parity before trusting the comparison.

## Limitations

- The comparison covers the specific case studies in the paper (aroma odor and human breath GC–MS data); generalization to other metabolomics/chemometrics settings is stated as potential ('NPFimg is potentially applicable to diverse metabolomics/chemometrics') but not benchmarked here.
- Error-rate assessment depends on ground-truth/reference assignments for which signals and marker identifications are erroneous; the article does not supply explicit threshold values for these judgments.
- NPFimg is open source under the MIT license, but its README notes documentation is incomplete ('I will update soon'), so reproducing the full pipeline may require consulting the paper's manual script.
- The claimed advantage rests on NPFimg avoiding the conventional peak-picking process, which 'suffers from false peak detections'; conclusions may not transfer to data where peak picking is not the dominant error source.

## Evidence
