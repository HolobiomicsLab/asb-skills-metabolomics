---
name: peak-shape-manual-annotation
description: Use when an automated feature-detection output (mzLearn, XCMS, ASARI)
  must be scored for accuracy and you need peak-shape-based ground-truth labels for
  true-positive and false-positive rates.
when_to_use_negative:
- The goal is de-isotoping, MS/MS mapping, or compound identification — mzLearn output
  stays at the ion MS1 level and downstream annotation workflows must be used instead.
- A fully automated, reproducible quality filter is needed — manual peak-shape annotation
  is subjective and is meant as ground-truth, not a production filter.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzLearn
  role: Provides the detected MS1 signal set whose features are manually annotated
    by peak shape to establish ground-truth TP/FP rates (published TP 89.0%, FP 12.5%).
  repo: github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Alternative peak-detection output whose features are scored against the same
    peak-shape ground truth (TP 77.4%, FP 17.3%).
- name: ASARI
  role: Third peak-detection output benchmarked against manual peak-shape labels (TP
    49.6%, FP 18.8%).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:f29ff6aa8225ac3da7d39052418e24a4437a61ae470630fcce1104ea835bdee4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - ASARI
  - XCMS
  - mzLearn
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-shape-manual-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Manually label detected LC/MS MS1 signals by inspecting extracted-ion chromatogram peak shapes, producing a ground-truth (true/false positive) call that benchmarks mzLearn, XCMS, and ASARI signal-detection performance.

## When to use

Use when an automated feature-detection output (mzLearn, XCMS, ASARI) must be scored for accuracy and you need peak-shape-based ground-truth labels for true-positive and false-positive rates.

## When NOT to use

- The goal is de-isotoping, MS/MS mapping, or compound identification — mzLearn output stays at the ion MS1 level and downstream annotation workflows must be used instead.
- A fully automated, reproducible quality filter is needed — manual peak-shape annotation is subjective and is meant as ground-truth, not a production filter.

## Inputs

- mzML files of LC/MS runs from the 15 public targeted-metabolomics benchmark datasets
- Detected MS1 feature tables (median rt and m/z values with per-sample normalized intensities) from mzLearn, XCMS, or ASARI

## Outputs

- Peak-shape ground-truth labels per detected feature (true positive / false positive)
- Per-tool, per-dataset TP-rate and FP-rate tabulation (benchmark_results.csv)

## How to apply

For each detected MS1 ion feature (defined by median m/z and retention time), extract the chromatographic peak across the samples of the benchmark mzML dataset and visually assess whether the apex is sharp, well-formed, and reproducible rather than a flat or noisy shoulder — this peak-shape judgement is the article's ground-truth standard. Annotate each feature as a true positive (well-formed peak) or false positive (noise, shoulder, or spurious signal), then compare these labels against the tool's detection call. On the 15 public targeted-metabolomics datasets this yields TP rates of 89.0% (mzLearn), 77.4% (XCMS), and 49.6% (ASARI) and FP rates of 12.5%, 17.3%, and 18.8% respectively. Tabulate the annotations per tool per dataset so per-tool TP/FP rates can be computed and validated against the published averages.

## Related tools

- **mzLearn** (Provides the detected MS1 signal set whose features are manually annotated by peak shape to establish ground-truth TP/FP rates (published TP 89.0%, FP 12.5%).) — github.com/ReviveMed/mzEmbed
- **XCMS** (Alternative peak-detection output whose features are scored against the same peak-shape ground truth (TP 77.4%, FP 17.3%).)
- **ASARI** (Third peak-detection output benchmarked against manual peak-shape labels (TP 49.6%, FP 18.8%).)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Reproduced average detected-signal counts match the published values: mzLearn 11,442, XCMS 7,100, ASARI 4,655 across the 15 datasets.
- Computed TP rates match 89.0% / 77.4% / 49.6% for mzLearn / XCMS / ASARI, and FP rates match 12.5% / 17.3% / 18.8%.
- Every annotated feature carries an explicit TP/FP call; no feature is left unlabelled.
- Annotation decisions are submitted to expert review and confirmed by independent manual peak-shape labelling, as the article describes.
- Per-dataset TP/FP counts, when averaged, are consistent with the tabulated benchmark_results.csv.

## Limitations

- Manual peak-shape annotation is subjective; the article notes verification was confirmed by expert review of manual labelling.
- Only applies to MS1-level ion features; it does not cover de-isotoping or MS/MS identification, which mzLearn does not perform.
- The mzLearn benchmarking context is limited to the 15 public targeted-metabolomics datasets in Supplementary Table 1; rates may not transfer to other cohorts or LC modes (the pre-trained model is HILIC positive mode only).

## Evidence

- [readme] Detected-feature definition used as annotation object: [evidence span withheld — anchor status: external_doc]
