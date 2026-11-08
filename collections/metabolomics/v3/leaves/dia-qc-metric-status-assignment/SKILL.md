---
name: dia-qc-metric-status-assignment
description: Use when per-file DIA QC metric values must be converted into categorical
  qualified/unqualified status labels and aggregated into overall LC and MS quality
  calls for downstream AI-model training or expert-agreement benchmarking.
when_to_use_negative:
- Input is DDA or SRM data — the skill is specific to DIA files; the article states
  prior QC pipelines for DDA/SRM cannot be directly applied to large-scale DIA experiments
- Raw vendor files (.d/.raw/.wiff) have not yet been converted to mzXML and analyzed
  with DIA-NN — metric extraction is a prerequisite
- You need absolute quantitative QC thresholds rather than binary qualified/unqualified
  calls — the skill outputs categorical status labels, not numeric metric cutoffs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: iDIA-QC
  role: 'Runs the full pipeline: reads raw DIA files (.d/.raw/.wiff), converts via
    msConvert, extracts the 15 metrics, and applies the trained machine-learning models
    to assign qualified/unqualified status per metric for LC and MS'
  repo: github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Analyzes DIA files (v1.7.15) to derive peptide and protein quantification
    results from which QC metrics are extracted
- name: msConvert
  role: Converts vendor raw files (.d/.raw/.wiff) to mzXML for metric extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:65f880de41c52146b784c0ae34810a820b1248bc18def6c62ed30b747b3b9377
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dia-qc-metric-status-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns a qualified/unqualified (pass/fail) status label to each of the 15 prioritized LC-MS QC metrics extracted from DIA raw files, either via majority voting over 21 expert raters or via the iDIA-QC machine-learning model, and aggregates per-metric labels into overall LC and MS qualified/unqualified evaluations.

## When to use

Use when per-file DIA QC metric values must be converted into categorical qualified/unqualified status labels and aggregated into overall LC and MS quality calls for downstream AI-model training or expert-agreement benchmarking.

## When NOT to use

- Input is DDA or SRM data — the skill is specific to DIA files; the article states prior QC pipelines for DDA/SRM cannot be directly applied to large-scale DIA experiments
- Raw vendor files (.d/.raw/.wiff) have not yet been converted to mzXML and analyzed with DIA-NN — metric extraction is a prerequisite
- You need absolute quantitative QC thresholds rather than binary qualified/unqualified calls — the skill outputs categorical status labels, not numeric metric cutoffs

## Inputs

- Per-rater expert annotation table covering the 2638 DIA files with 21 raters per file
- Extracted numerical QC metric values per DIA file (from mzXML and DIA-NN outputs)
- Instrument metadata mapping each file to one of 21 mass spectrometers across nine laboratories
- Identifiers of the 11 embedded technical replicates

## Outputs

- majority_vote_consensus_labels.csv — per-file per-metric consensus qualified/unqualified labels plus overall LC and MS status
- observed_agreement_stats.tsv — per-instrument and technical-replicate observed agreement statistics
- Per-file machine-learning output assigning qualified/unqualified evaluations to each metric (LC and MS predictions)

## How to apply

First extract the 15 prioritized QC metrics (from the over 140 candidate metrics compiled in the study) for each DIA file from mzXML and DIA-NN outputs. For expert-based labeling, load the per-rater annotation table covering the 2638 DIA files with pandas and apply majority voting across the 21 expert raters per metric to produce a consensus qualified/unqualified label; derive each file's overall LC and MS status from the per-metric consensus. For model-based labeling, feed the extracted metric vector into the iDIA-QC machine-learning model, which assigns a qualified/unqualified result to each metric. The expert-consensus labels serve as training labels for the AI model (2110 training files out of 2754 DIA files), with two separate models predicting LC and MS performance. Validate by checking that the median observed agreement across the 21 instruments falls within the reported 73–90% range and reaches 100% on the 11 embedded technical replicates.

## Related tools

- **iDIA-QC** (Runs the full pipeline: reads raw DIA files (.d/.raw/.wiff), converts via msConvert, extracts the 15 metrics, and applies the trained machine-learning models to assign qualified/unqualified status per metric for LC and MS) — github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Analyzes DIA files (v1.7.15) to derive peptide and protein quantification results from which QC metrics are extracted)
- **msConvert** (Converts vendor raw files (.d/.raw/.wiff) to mzXML for metric extraction)

## Examples

```
from iDIAQC import predict_status; labels = predict_status('out_dir/metrics.csv', instrument_type='timsTOF', model='MS')
```

## Evaluation signals

- Median observed agreement across the 21 instruments falls within the reported 73–90% range
- Median observed agreement on the 11 embedded technical replicates reaches 100%
- Per-file output contains a qualified/unqualified label for each of the 15 metrics plus aggregate LC and MS status
- Output CSV/TSV row count matches the number of DIA files processed (e.g., 2638 for the expert-consensus set, 2754 total)
- For the AI-model path, per-metric status assignments reproduce the expert-consensus labels with AUC > 0.91 on the independent validation set (n = 528)

## Limitations

- The study could not cover all LC and MS instruments, especially newly emerging ones, so the metric-to-status mapping may not generalize to unsupported hardware
- Expert majority voting is limited to the 21 raters; model-building details (hyperparameters, train/test split composition) are not fully described
- Generalization of the workflow to other QC and experimental samples requires further investigation

## Evidence

- [abstract] Reported expert agreement range as validation signal: [evidence span withheld — anchor status: not_found]
