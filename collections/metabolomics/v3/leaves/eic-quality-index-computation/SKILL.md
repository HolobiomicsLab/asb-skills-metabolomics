---
name: eic-quality-index-computation
description: Use when scoring extracted ion chromatogram (EIC) quality prior to peak
  detection in LC/MS metabolomics workflows, or when benchmarking the MCQ index against
  alternatives (e.g., the global zigzag index) on normal, high-background, or single-peak
  EIC profiles.
when_to_use_negative:
- The EICs to evaluate have high background signals or contain only a single peak
  — the paper shows MCQ cannot fairly evaluate these cases; use the EIC-related global
  zigzag index instead.
- The goal is evaluating detected chromatographic peak quality rather than EIC quality
  — that stage uses separate metrics (sharpness, Gaussian similarity, SNR, peak significance
  level, TPASR, local peak-related zigzag index) applied after peak detection.
- Selecting and aligning analyte components is the primary objective rather than general
  EIC quality assessment — although MCQ is suited to that narrower task, it is not
  a general EIC quality metric.
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0092
tools:
- name: Matlab
  role: Implementation language for the custom data processing program that computes
    the spike detection index, background detection index, and combined MCQ index
    over the synthetic EIC battery
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:a0bddc4c987fa34c1a943faa39a7733cf1c5abcd41994f8cf83d55f6b6cf8fbb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# eic-quality-index-computation

## Summary

Computes the mass chromatographic quality (MCQ) index for extracted ion chromatograms (EICs) by combining a spike detection index (original vs. smoothed signal similarity) and a background detection index (original vs. mean-subtracted signal similarity), yielding a bounded 0–1.0 quality score per EIC. It serves as the comparator baseline for the paper's proposed global zigzag index and is demonstrated to fail on high-background and single-peak EICs.

## When to use

Use when scoring extracted ion chromatogram (EIC) quality prior to peak detection in LC/MS metabolomics workflows, or when benchmarking the MCQ index against alternatives (e.g., the global zigzag index) on normal, high-background, or single-peak EIC profiles.

## When NOT to use

- The EICs to evaluate have high background signals or contain only a single peak — the paper shows MCQ cannot fairly evaluate these cases; use the EIC-related global zigzag index instead.
- The goal is evaluating detected chromatographic peak quality rather than EIC quality — that stage uses separate metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) applied after peak detection.
- Selecting and aligning analyte components is the primary objective rather than general EIC quality assessment — although MCQ is suited to that narrower task, it is not a general EIC quality metric.

## Inputs

- Extracted ion chromatogram (EIC) signal vectors extracted from an LC/MS profile dataset (e.g., from the four-module Matlab pipeline: acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering)
- Deterministic synthetic EIC battery with three profile types (normal, high-background, single-peak) generated with fixed parameters

## Outputs

- Per-case results table mcq_index_values.csv with columns EIC_ID, EIC type, MCQ_index
- One MCQ index value per EIC_ID, bounded in [0, 1.0]
- Intermediate spike detection index and background detection index values per EIC

## How to apply

First generate a deterministic synthetic EIC battery with three profile types — normal, high-background, and single-peak — using fixed parameters so MCQ values are reproducible. For each EIC, implement the spike detection index as the similarity between the original EIC and its smoothed version, and the background detection index as the similarity between the original EIC and its mean-subtracted version. Combine both similarity indexes into the MCQ index, which is by definition bounded in the dynamic range 0 to 1.0. Run the MCQ function over every EIC in the battery and record one MCQ value per EIC_ID. Write the per-case results table with columns EIC_ID, EIC type, and MCQ_index to mcq_index_values.csv, and validate against the paper's finding that MCQ cannot fairly evaluate high-background or single-peak EICs.

## Related tools

- **Matlab** (Implementation language for the custom data processing program that computes the spike detection index, background detection index, and combined MCQ index over the synthetic EIC battery)

## Examples

```
mcq = spikeIdx(eic, smooth(eic)) .* bgIdx(eic, mean(eic)); writetable(table(eicID, eicType, mcq, 'VariableNames', {'EIC_ID','EIC_type','MCQ_index'}), 'mcq_index_values.csv'); assert(all(mcq >= 0 & mcq <= 1.0))
```

## Evaluation signals

- All computed MCQ values must lie within [0, 1.0]
- The normal EIC must score a high MCQ index
- The high-background and single-peak EICs must produce relatively low MCQ indices, consistent with the paper's finding that MCQ cannot fairly evaluate these EIC types
- mcq_index_values.csv contains exactly one MCQ value per EIC_ID with columns EIC_ID, EIC type, and MCQ_index
- Re-running the battery with fixed synthetic EIC parameters must reproduce identical MCQ values

## Limitations

- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak, limiting its use as a general EIC quality metric
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository or license is provided
- Optimal cutoff thresholds for each quality metric remain an unsolved challenge, so MCQ thresholding for EIC filtering must be set empirically

## Evidence

- [abstract] MCQ cannot fairly evaluate high-background or single-peak EICs: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [methods] Custom Matlab pipeline with four sequential modules including EIC quality evaluation and filtering: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [methods] EIC quality evaluation and filtering is a defined pipeline stage: "2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [discussion] Cutoff thresholds are hard to set optimally: "setting an optimal cutoff threshold for each metric still is a challenge task"
