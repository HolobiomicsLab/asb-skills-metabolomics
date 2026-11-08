---
name: relative-standard-deviation-qc-metrics
description: Use when you need to quantify residual intensity drift in LC/MS untargeted
  metabolomics after normalization, by computing per-feature RSD across withheld QC
  sample injections on a multi-sample feature matrix.
when_to_use_negative:
- Your dataset already has a standardized QC sample scheme that normalizes drift correction
  internally (i.e., the paper notes public repositories often lack standardized QC
  samples, so this comparison only makes sense when QC samples are absent or withheld)
- You want to evaluate signal detection (TP/FP rates) rather than post-normalization
  intensity stability — use mzLearn detection benchmarks instead
- The input is already a pre-normalized, low-RSD feature table and you are not comparing
  multiple normalization treatments
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzLearn
  role: Generates the ion-feature matrices and performs data-driven drift correction
    with internally generated synthetic QC samples; its output intensity matrices
    are what RSD is computed on
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:f995cc29cadb65a33f402fe8a15d55d93aa9df6515b0df6ad0f85081ca4e6f20
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - mzLearn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# relative-standard-deviation-qc-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes relative standard deviation (RSD/CV×100) of ion-feature intensities across hidden QC injections to benchmark drift-correction quality of normalization treatments such as synthetic QC, experimental QC, TIC, and raw intensities on mzLearn feature matrices.

## When to use

Use when you need to quantify residual intensity drift in LC/MS untargeted metabolomics after normalization, by computing per-feature RSD across withheld QC sample injections on a multi-sample feature matrix.

## When NOT to use

- Your dataset already has a standardized QC sample scheme that normalizes drift correction internally (i.e., the paper notes public repositories often lack standardized QC samples, so this comparison only makes sense when QC samples are absent or withheld)
- You want to evaluate signal detection (TP/FP rates) rather than post-normalization intensity stability — use mzLearn detection benchmarks instead
- The input is already a pre-normalized, low-RSD feature table and you are not comparing multiple normalization treatments

## Inputs

- mzLearn ion-feature matrices (CSV of floats, shape [n_samples, n_features], n_samples > 150) for normalization-evaluation datasets
- Identity of hidden experimental QC injections per dataset (to exclude from processing and use only for RSD evaluation)
- Treatment labels: synthetic QC normalization, experimental QC normalization, TIC normalization, raw intensities

## Outputs

- qc_rsd_results.csv — per-dataset RSD values (CV×100) tabulated side by side for the four normalization conditions
- Mean RSD summary values across datasets for each condition

## How to apply

Load the mzLearn ion-feature matrices (each with more than 150 samples) for the normalization-evaluation datasets, keeping the experimental QC injections hidden from all processing. Run mzLearn's data-driven drift correction using internally generated synthetic QC samples, and in parallel apply the comparator treatments: correction using the withheld experimental QC samples, total-ion-current (TIC) normalization, and no normalization (raw intensities). For each treatment, compute RSD (coefficient of variation × 100) of feature intensities across the hidden QC injections per dataset. Aggregate the per-dataset RSDs into a side-by-side table (qc_rsd_results.csv) so the four conditions can be compared directly.

## Related tools

- **mzLearn** (Generates the ion-feature matrices and performs data-driven drift correction with internally generated synthetic QC samples; its output intensity matrices are what RSD is computed on)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Mean RSD for synthetic QC normalization should fall near 17%, matching the reported benchmark in Fig. 3c / Supplementary Table 4
- Mean RSD for experimental QC normalization should fall near 16%
- Mean RSD for TIC normalization should fall near 23%
- Mean RSD for raw (no normalization) intensities should fall near 25%
- The synthetic-QC RSD should be comparable to (within ~1–2 percentage points of) the experimental-QC RSD, confirming synthetic QC can correct drift without real QC injections

## Limitations

- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, so experimental-QC comparison may not always be available
- RSD evaluation depends on having enough samples (>150) per dataset to be representative
- mzLearn's iterative drift-correction can take up to 2.5 days on a single machine, which limits rapid iteration on the RSD benchmark
- The current pre-trained mzLearn model is specific to HILIC positive mode; RSD behavior on other LC modes is not established

## Evidence

- [supplementary] Benchmark RSD values for the four conditions: [evidence span withheld — anchor status: not_found]
- [readme] mzLearn workflow and data input format: [evidence span withheld — anchor status: not_found]
