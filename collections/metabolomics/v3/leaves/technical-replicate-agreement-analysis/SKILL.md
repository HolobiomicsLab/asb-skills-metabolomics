---
name: technical-replicate-agreement-analysis
description: Use when per-file human QC annotations from multiple raters must be reduced
  to consensus labels and quantified for inter-rater agreement, especially on embedded
  technical replicates.
when_to_use_negative:
- Input is already a single-label ground truth (e.g. AI model predictions) — this
  skill requires multiple independent rater labels to form a majority vote; applying
  it to model output conflates prediction with consensus.
- Data are DDA files or targeted SRM assays rather than DIA acquisition — the annotation
  consensus and replicate structure described here were established for the 2754 DIA
  QC files from 21 multivendor instruments, and the paper notes SRM-based QC tools
  cannot be applied directly to large-scale experiments.
- Fewer than three raters per file are available — majority voting and meaningful
  observed-agreement statistics need a rater panel comparable to the 21-expert panel;
  with a single annotator the consensus degenerates to one opinion.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: GUI and pipeline that produced the 15 QC metrics and file-level quality annotations
    whose expert labels are reduced to consensus and agreement statistics here
  repo: https://github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:aeefd4c3e755f354613ea3bcba7ad385ebace340670c455181c3679cbeea735b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  repo_url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - iDIA-QC
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# technical-replicate-agreement-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes how consistently multiple expert annotators (and embedded technical replicates) agree on qualified/unqualified QC labels for DIA mass-spectrometry files, using majority-vote consensus as the reference label. Validates that the annotation layer of the iDIA-QC workflow is reproducible before that layer is used to train or benchmark AI QC models.

## When to use

Use when per-file human QC annotations from multiple raters must be reduced to consensus labels and quantified for inter-rater agreement, especially on embedded technical replicates.

## When NOT to use

- Input is already a single-label ground truth (e.g. AI model predictions) — this skill requires multiple independent rater labels to form a majority vote; applying it to model output conflates prediction with consensus.
- Data are DDA files or targeted SRM assays rather than DIA acquisition — the annotation consensus and replicate structure described here were established for the 2754 DIA QC files from 21 multivendor instruments, and the paper notes SRM-based QC tools cannot be applied directly to large-scale experiments.
- Fewer than three raters per file are available — majority voting and meaningful observed-agreement statistics need a rater panel comparable to the 21-expert panel; with a single annotator the consensus degenerates to one opinion.

## Inputs

- deposited per-rater annotation table for the 2754 DIA files (2638 analyzed) with qualified/unqualified labels per QC metric per expert
- instrument metadata mapping each file to one of the 21 mass spectrometers (Orbitrap, TripleTOF, timsTOF Pro series)
- identification of the 11 embedded technical replicates
- list of the 15 prioritized QC metrics used for LC and MS assessment

## Outputs

- majority_vote_consensus_labels.csv — per-file consensus qualified/unqualified labels per metric plus overall LC and MS status
- observed_agreement_stats.tsv — per-instrument observed agreement and agreement on the 11 embedded technical replicates
- validation report asserting median agreement falls in the 73–90% range and replicate agreement reaches 100%

## How to apply

Load the deposited per-rater annotation table covering the 2638 DIA files and join each file to its instrument metadata for the 21 mass spectrometers studied. For each file and each of the 15 QC metrics, apply majority voting across the 21 expert raters to assign a qualified/unqualified consensus label; aggregate the per-metric consensus labels to derive overall LC and MS qualified/unqualified status per file. Compute observed agreement — the proportion of raters whose label matches the consensus — per instrument, and separately on the 11 embedded technical replicates. Write per-file consensus labels to majority_vote_consensus_labels.csv and the per-instrument and replicate agreement statistics to observed_agreement_stats.tsv. Validate that the median observed agreement across instruments falls within the reported 73–90% range and that median agreement on the embedded technical replicates reaches 100%; these thresholds confirm the annotation layer is reproducible enough to serve as ground truth for the downstream AI model.

## Related tools

- **iDIA-QC** (GUI and pipeline that produced the 15 QC metrics and file-level quality annotations whose expert labels are reduced to consensus and agreement statistics here) — https://github.com/guomics-lab/iDIA-QC

## Evaluation signals

- Median observed agreement across the 21 instruments falls within the reported 73–90% range.
- Median observed agreement on the 11 embedded technical replicates reaches 100%.
- majority_vote_consensus_labels.csv contains one consensus row per DIA file with both per-metric labels and aggregated overall LC/MS status.
- observed_agreement_stats.tsv contains one agreement row per instrument (21 rows) plus a separate replicate-agreement entry.
- Every file in the annotation table appears in exactly one consensus row and every rater vote is counted — no files silently dropped by the join to instrument metadata.

## Limitations

- Expert annotation is reasonably reliable but not perfect — median agreement of 73–90% means individual raters diverge from consensus on roughly 10–27% of files, so consensus labels are probabilistic ground truth rather than error-free labels.
- The 15 QC metrics and the qualified/unqualified task were tuned for DIA files from timsTOF, TripleTOF, and Orbitrap instruments; agreement statistics may not transfer to other instrument families.
- Agreement on the 11 embedded technical replicates reaching 100% reflects only that narrow replicate set, not general rater consistency.

## Evidence
