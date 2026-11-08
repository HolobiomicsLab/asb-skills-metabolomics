---
name: chromatogram-smoothing-similarity-analysis
description: Use when an LC/MS EIC must be scored for quality via smoothing- and background-similarity
  measures, or when benchmarking a comparator EIC metric against a proposed alternative
  (e.g. the global zigzag index).
when_to_use_negative:
- EICs with high background signals or containing only a single peak need a fair quality
  verdict — the paper explicitly states the MCQ index 'cannot fairly evaluate' these
  cases; use the global zigzag index instead
- Peak-level (not EIC-level) quality evaluation — the paper's six peak quality metrics
  (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag
  index) apply to detected chromatographic peaks, not whole EICs
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Development environment used by the authors to implement the four-module data
    processing program, including the MCQ index computation over EICs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:63f98e40f248d132df1cb21cb531a0bea9c3f584781116801da701c9c809f615
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chromatogram-smoothing-similarity-analysis

## Summary

Implements the MCQ (mass chromatographic quality) index for extracted ion chromatograms (EICs) by quantifying similarity between an EIC and its smoothed version (spike detection) and its mean-subtracted version (background detection), then combining the two similarities into a 0–1.0 quality score. The paper shows this index works for selecting/aligning analyte components but fails to fairly evaluate EICs with high background signals or only a single peak.

## When to use

Use when an LC/MS EIC must be scored for quality via smoothing- and background-similarity measures, or when benchmarking a comparator EIC metric against a proposed alternative (e.g. the global zigzag index).

## When NOT to use

- EICs with high background signals or containing only a single peak need a fair quality verdict — the paper explicitly states the MCQ index 'cannot fairly evaluate' these cases; use the global zigzag index instead
- Peak-level (not EIC-level) quality evaluation — the paper's six peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) apply to detected chromatographic peaks, not whole EICs

## Inputs

- Extracted ion chromatogram (EIC) profiles (synthetic deterministic battery of normal, high-background, and single-peak profile types, or real EICs extracted from an LC/MS profile dataset)
- Smoothing-filter parameterization for the smoothed (spike-detection) version of each EIC
- Mean-subtracted version of each EIC for background detection

## Outputs

- Per-case results table with columns EIC_ID, EIC type, and MCQ_index written to mcq_index_values.csv
- One MCQ value per EIC, bounded in [0, 1.0]

## How to apply

Generate a deterministic synthetic EIC battery covering three profile types — normal, high-background, and single-peak — with fixed parameters so values are reproducible. For each EIC compute the spike detection index as the similarity between the original EIC and its smoothed version, and the background detection index as the similarity between the original EIC and its mean-subtracted version; combine the two into the MCQ index, which is by definition bounded in the dynamic range 0 to 1.0. Run the MCQ function over every EIC in the battery, recording one MCQ value per EIC_ID, and write the per-case results table (EIC_ID, EIC type, MCQ_index) to mcq_index_values.csv. Rationale: the paper's finding that the MCQ index cannot fairly evaluate high-background or single-peak EICs means those cases should score relatively low, whereas a normal zigzag EIC scores high.

## Related tools

- **Matlab** (Development environment used by the authors to implement the four-module data processing program, including the MCQ index computation over EICs)

## Examples

```
% Matlab: run MCQ over the synthetic EIC battery; MCQ = spikeSim .* bgSim (in [0,1.0])
T = table(eic_ids, eic_types, MCQ_values, 'VariableNames', {'EIC_ID','EIC_type','MCQ_index'}); writetable(T, 'mcq_index_values.csv');
```

## Evaluation signals

- All MCQ values must lie in the dynamic range [0, 1.0]
- The normal EIC must score a high MCQ index
- The high-background EIC and the single-peak EIC must produce relatively low MCQ indices, reproducing the paper's finding that MCQ cannot fairly evaluate these scenarios
- mcq_index_values.csv must contain exactly one MCQ row per EIC_ID with the columns EIC_ID, EIC type, MCQ_index
- Re-running the synthetic battery with fixed parameters must yield identical MCQ values (reproducibility)

## Limitations

- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak, limiting its use as a general EIC quality metric
- Setting an optimal cutoff threshold for each quality metric is still a challenging task
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository or software license is provided

## Evidence

- [abstract] Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak.: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [methods] Matlab program modules: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [intro] EIC metric investigation: "we investigated potential metrics for evaluating extracted EIC and detected chromatographic peak quality."
- [discussion] Cutoff challenge: "setting an optimal cutoff threshold for each metric still is a challenge task"
