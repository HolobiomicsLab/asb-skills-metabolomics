---
name: expert-annotation-majority-voting
description: Use when multiple independent raters have each scored the same set of
  files (e.g. qualified/unqualified per instrument run) and you need a reproducible
  ground-truth label or agreement statistics before training or benchmarking.
when_to_use_negative:
- Raters produced scores on a continuous scale rather than categorical qualified/unqualified
  votes, so majority voting is ill-defined without thresholding first
- Only one annotator is available per file, making agreement statistics meaningless
- You need fine-grained per-metric disagreement analysis (e.g. which specific QC metric
  caused divergence) rather than a single consensus label
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDIA-QC
  role: generates the 15 numerical QC metrics that experts assessed; the consensus
    labels derived here are the ground truth the iDIA-QC ML model was trained on
  repo: github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: produces the peptide/protein quantification results from which the 15 metrics
    were extracted, forming the feature context each expert rated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:7fafd0175ada9a4fb5f2bcdb1dc1ebe5bb306d88f195f5b78b6a23238738bea5
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# expert-annotation-majority-voting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregate manual QC assessments from multiple domain experts (e.g. 21 experts scoring 2754 DIA files) into a single consensus label per file via majority voting, and quantify inter-expert observed agreement to establish ground-truth annotations for downstream model training and validation.

## When to use

Use when multiple independent raters have each scored the same set of files (e.g. qualified/unqualified per instrument run) and you need a reproducible ground-truth label or agreement statistics before training or benchmarking.

## When NOT to use

- Raters produced scores on a continuous scale rather than categorical qualified/unqualified votes, so majority voting is ill-defined without thresholding first
- Only one annotator is available per file, making agreement statistics meaningless
- You need fine-grained per-metric disagreement analysis (e.g. which specific QC metric caused divergence) rather than a single consensus label

## Inputs

- per-rater annotation table (e.g. CSV) mapping each DIA file ID to the qualified/unqualified vote of each expert
- instrument metadata mapping each file to one of the 21 mass spectrometers
- list of technical-replicate file IDs (e.g. the 11 embedded replicates)
- number of expert raters per file (here 21)

## Outputs

- majority_vote_consensus_labels.csv — per-file consensus qualified/unqualified label
- observed_agreement_stats.tsv — per-instrument and per-replicate observed agreement (proportion of raters matching consensus)
- overall LC and MS qualified/unqualified status per file derived from per-metric votes

## How to apply

Load the per-rater annotation table — here, 21 experts manually assessed the quality of 2754 DIA files based on 15 prioritized metrics — with pandas, joining each file to its instrument metadata (21 mass spectrometers across nine laboratories over 31 months). For each file and each metric, apply majority voting across the 21 expert raters to assign a qualified/unqualified consensus label, then aggregate per-metric votes into overall LC and MS qualified/unqualified status per file. Separately compute observed agreement as the proportion of raters matching the consensus label, aggregated per instrument and on any embedded technical replicates (the study had 11 embedded replicates among the 2638 DIA files). Rationale: majority voting reduces single-rater bias and yields a stable label that the article's ML model (AUC 0.91 for LC, 0.97 for MS on an independent validation set of n = 528) was trained against, so consensus reproducibility must be verified before treating those labels as ground truth.

## Related tools

- **iDIA-QC** (generates the 15 numerical QC metrics that experts assessed; the consensus labels derived here are the ground truth the iDIA-QC ML model was trained on) — github.com/guomics-lab/iDIA-QC
- **DIA-NN** (produces the peptide/protein quantification results from which the 15 metrics were extracted, forming the feature context each expert rated)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('per_rater_annotations.csv'); consensus = df.groupby('file_id').agg(lambda x: x.value_counts().idxmax()); consensus.to_csv('majority_vote_consensus_labels.csv')"
```

## Evaluation signals

- Median observed agreement across the 21 instruments falls within the reported 73–90% range
- Median observed agreement on the 11 embedded technical replicates reaches 100%
- Every file in the 2754-DIA-file set receives exactly one consensus label (no ties or nulls)
- Per-metric vote counts sum to the number of raters for each file-metric pair
- Aggregated per-metric consensus labels reproduce the overall LC/MS status reported in the study for spot-checked files

## Limitations

- Manual annotation is inherently subjective; the median observed agreement (73–90%) means up to ~27% of files had minority dissent
- Consensus labels are only as good as the 15 prioritized metrics; the article notes over 140 candidate metrics were narrowed to 15, so metric choice bounds what experts could assess
- The study could not cover all LC and MS instruments, particularly newly emerging ones, so consensus may not generalize

## Evidence
