---
name: hidden-qc-drift-assessment
description: 'Use when evaluating whether a synthetic-QC drift-correction approach
  (e.g. mzLearn) is an adequate substitute for experimental QC samples: when you must
  compare normalization strategies on the same feature matrix without exposing QC
  samples during correction.'
when_to_use_negative:
- When the goal is to correct drift for downstream analysis (i.e. you want normalization
  applied to study samples rather than to benchmark correction quality on withheld
  QC)
- When no experimental QC injections exist in the dataset to serve as an independent
  held-out drift benchmark (the article notes public metabolomics repositories often
  lack standardized QC samples)
- When evaluating drift correction on MS/MS-level or de-isotoped compound tables rather
  than the ion MS1 feature matrices mzLearn outputs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzLearn
  role: Runs data-driven drift correction using internally generated synthetic QC
    samples on the ion-feature matrices to produce the synthetic-QC normalization
    condition being benchmarked.
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:2c4f86894e0cfa6c68c2d7230bed766d0f83bef0dff61a090ffe3afa5c1906cd
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
yes_rate: 0.875
n_claims: 8
---

# hidden-qc-drift-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks drift-correction strategies on LC/MS feature-intensity matrices by withholding experimental QC injections entirely and comparing their relative standard deviation (RSD) in the hidden QC samples, quantifying how well data-driven synthetic QC normalization from mzLearn matches experimental QC, TIC, and raw normalization.

## When to use

Use when evaluating whether a synthetic-QC drift-correction approach (e.g. mzLearn) is an adequate substitute for experimental QC samples: when you must compare normalization strategies on the same feature matrix without exposing QC samples during correction.

## When NOT to use

- When the goal is to correct drift for downstream analysis (i.e. you want normalization applied to study samples rather than to benchmark correction quality on withheld QC)
- When no experimental QC injections exist in the dataset to serve as an independent held-out drift benchmark (the article notes public metabolomics repositories often lack standardized QC samples)
- When evaluating drift correction on MS/MS-level or de-isotoped compound tables rather than the ion MS1 feature matrices mzLearn outputs

## Inputs

- mzLearn ion-feature matrices (samples x feature intensities) for each normalization-evaluation dataset (more than 150 samples each)
- sample metadata distinguishing experimental QC injections from study samples

## Outputs

- qc_rsd_results.csv: per-dataset RSD (CV x 100) of feature intensities across hidden QC injections for synthetic QC, experimental QC, TIC, and raw normalization conditions

## How to apply

Load the mzLearn ion-feature matrices for each dataset (six normalization-evaluation datasets with more than 150 samples each), keeping the experimental QC injections withheld (hidden) from all processing. Run mzLearn's data-driven drift correction using its internally generated synthetic QC samples. Apply the comparator treatments on the same matrices: correction using the withheld experimental QC samples, total-ion-current (TIC) normalization, and raw (no normalization). For each treatment, compute the relative standard deviation (coefficient of variation x 100) of feature intensities across the hidden QC injections per dataset, then aggregate the per-dataset RSDs and tabulate the four conditions side by side into qc_rsd_results.csv. The rationale is that hidden QC RSD directly reflects residual signal-intensity drift; synthetic QC normalization achieved 17% RSD, comparable to 16% with experimental QC, versus 23% for TIC and 25% for raw normalization, so lower hidden-QC RSD indicates better drift removal without needing QC injections at correction time.

## Related tools

- **mzLearn** (Runs data-driven drift correction using internally generated synthetic QC samples on the ion-feature matrices to produce the synthetic-QC normalization condition being benchmarked.) — github.com/ReviveMed/mzEmbed

## Examples

```
python -m mz_learn.normalize --input features.csv --qc hidden --method synthetic_qc --rsd_out qc_rsd_results.csv
```

## Evaluation signals

- Mean RSD across hidden QC samples matches the reported benchmarks: 17% (synthetic QC), 16% (experimental QC), 23% (TIC), 25% (raw), as in Fig. 3c / Supplementary Table 4
- Per-dataset RSDs are computed only on feature intensities for hidden QC injections, never on study samples
- qc_rsd_results.csv tabulates all four normalization conditions side by side per dataset
- Experimental QC injections remain withheld (not used in correction) throughout processing

## Limitations

- Public metabolomics repositories often lack standardized QC samples, limiting this benchmark's applicability to studies without QC injections
- mzLearn's iterative learning is computationally intensive — analysis can take up to 2.5 days on a single machine for large datasets
- mzLearn outputs ion MS1-level features only; drift assessment does not cover MS/MS identification or de-isotoped compound-level data

## Evidence

- [results] synthetic QC normalization achieved 17% RSD in hidden QC samples, comparable to 16% RSD when normalizing with experimental QC samples: [evidence span withheld — anchor status: not_found]
- [results] relative standard deviation of feature intensities across the hidden QC injections: [evidence span withheld — anchor status: not_found]
- [readme] no prior knowledge or QC samples are required: [evidence span withheld — anchor status: external_doc]
