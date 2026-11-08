---
name: lc-ms-qc-consensus-labeling
description: Use when multi-rater quality annotations of LC-MS files (e.g., qualified/unqualified
  per metric or per file) must be collapsed to a single consensus label and reproducibility
  (observed agreement) must be quantified per instrument.
when_to_use_negative:
- Input consists of raw vendor files (.d/.raw/.wiff) rather than existing expert annotations
  — the conversion and metric-extraction workflow applies, not consensus labeling.
- Only a single annotator's assessment is available — majority voting and observed-agreement
  statistics require ≥ 2 raters.
- Goal is to train or evaluate the AI model itself rather than to derive or audit
  the ground-truth consensus labels.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: iDIA-QC
  role: Source of the 15 prioritized QC metrics and the expert-inspection framework
    that the consensus labels validate; Python GUI for DIA file quality evaluation
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:46959dd47f7ecb29fb320a4c6f9b6fdcb1847b5f8260265eb58f3a3d03213fcc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - iDIA-QC
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# lc-ms-qc-consensus-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the majority-vote consensus labeling of LC/MS file quality from multi-expert annotations of DIA QC files and quantifies inter-rater observed agreement, establishing the ground-truth labels used to train iDIA-QC's AI models.

## When to use

Use when multi-rater quality annotations of LC-MS files (e.g., qualified/unqualified per metric or per file) must be collapsed to a single consensus label and reproducibility (observed agreement) must be quantified per instrument.

## When NOT to use

- Input consists of raw vendor files (.d/.raw/.wiff) rather than existing expert annotations — the conversion and metric-extraction workflow applies, not consensus labeling.
- Only a single annotator's assessment is available — majority voting and observed-agreement statistics require ≥ 2 raters.
- Goal is to train or evaluate the AI model itself rather than to derive or audit the ground-truth consensus labels.

## Inputs

- Deposited per-rater annotation table for DIA QC files (manual qualified/unqualified assessments by 21 experts)
- Instrument metadata mapping each file to one of 21 mass spectrometers
- Identifiers for embedded technical replicate pairs

## Outputs

- majority_vote_consensus_labels.csv (per-file consensus qualified/unqualified labels for LC and MS)
- observed_agreement_stats.tsv (per-instrument and per-replicate observed agreement statistics)

## How to apply

Load the deposited per-rater annotation table covering the DIA files with pandas and join each file to its instrument metadata (21 mass spectrometers across nine laboratories). For each file and each of the prioritized QC metrics, apply majority voting across the expert raters to assign a qualified/unqualified label, then aggregate per-metric labels to derive overall LC and MS qualified/unqualified status. Compute the observed agreement — the proportion of raters matching the consensus label — per instrument and separately on the embedded technical replicates. Write per-file consensus labels to a CSV and per-instrument/replicate agreement statistics to a TSV. Validate that the median observed agreement across instruments falls in the reported 73–90% range and that replicate agreement reaches 100%.

## Related tools

- **iDIA-QC** (Source of the 15 prioritized QC metrics and the expert-inspection framework that the consensus labels validate; Python GUI for DIA file quality evaluation) — github.com/guomics-lab/iDIA-QC

## Examples

```
import pandas as pd; df = pd.read_csv('expert_annotations.csv'); consensus = df.groupby('file').agg(lambda c: c.mode()[0]); consensus.to_csv('majority_vote_consensus_labels.csv')
```

## Evaluation signals

- Median observed agreement across the 21 instruments must fall within the reported 73–90% range
- Median observed agreement on the 11 embedded technical replicates must reach 100%
- Every file in the consensus table must carry both an LC and an MS qualified/unqualified status
- Per-file consensus label counts must match the number of raters contributing to each annotation

## Limitations

- Observed agreement is a proportion, not a chance-corrected statistic (e.g., Cohen's kappa); high baseline prevalence of 'qualified' labels could inflate agreement values.
- Consensus labels derived here are the ground truth for iDIA-QC's AI models; errors in expert judgment propagate into model training if not audited.
- The 21 experts assessed 2754 DIA files based on 15 metrics — annotations reflect human perception of those metrics, not instrument ground truth.

## Evidence

- [methods] Majority voting across expert raters derives per-metric consensus labels: [evidence span withheld — anchor status: not_found]
- [results] Observed agreement quantified per instrument and on technical replicates: [evidence span withheld — anchor status: not_found]
- [results] Expected agreement range from source card: [evidence span withheld — anchor status: not_found]
