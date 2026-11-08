---
name: inter-rater-agreement-statistics
description: Use when multiple independent annotators have labeled the same set of
  items (e.g. qualified/unqualified QC calls per file) and you need reproducibility
  statistics and consensus labels before training a supervised model.
when_to_use_negative:
- Input is already a single ground-truth label set with no multiple raters — there
  is no inter-rater signal to compute
- Raters labeled disjoint subsets of files rather than the same items, so majority
  voting across raters per item is undefined
- The task is automatic QC prediction from raw files, not human-label reproducibility
  assessment
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: Standalone GUI platform whose machine learning models consume the expert-consensus
    labels produced by this agreement analysis as training ground truth
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:834ba677f2bb8d55229499ee4a544a8cb0def731e9beba54d215f997363274b8
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# inter-rater-agreement-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how reliably human experts agree on mass-spectrometry QC labels by majority-voting across 21 raters and computing observed agreement per instrument, as done for the 2638 DIA files in the iDIA-QC study.

## When to use

Use when multiple independent annotators have labeled the same set of items (e.g. qualified/unqualified QC calls per file) and you need reproducibility statistics and consensus labels before training a supervised model.

## When NOT to use

- Input is already a single ground-truth label set with no multiple raters — there is no inter-rater signal to compute
- Raters labeled disjoint subsets of files rather than the same items, so majority voting across raters per item is undefined
- The task is automatic QC prediction from raw files, not human-label reproducibility assessment

## Inputs

- Per-rater annotation table (one row per file × rater × QC metric, qualified/unqualified) covering the 2638 DIA files
- Instrument metadata mapping each file to one of the 21 mass spectrometers (Orbitrap, TripleTOF, timsTOF Pro series)
- List of the 15 prioritized QC metrics used for manual assessment

## Outputs

- majority_vote_consensus_labels.csv — per-file consensus qualified/unqualified labels per metric and aggregated LC/MS status
- observed_agreement_stats.tsv — per-instrument and technical-replicate observed agreement percentages

## How to apply

Load the deposited per-rater annotation table covering the 2638 DIA files and join each file to its instrument metadata (21 mass spectrometers across nine laboratories). For each of the 15 prioritized QC metrics, apply majority voting across the 21 expert raters to assign a qualified/unqualified consensus label per file, then aggregate the per-metric consensus labels to derive the overall LC and MS qualified/unqualified status. Compute observed agreement as the proportion of raters matching the consensus label, reported separately per instrument and on the 11 embedded technical replicates. Validation: median observed agreement across instruments must fall in the reported 73–90% range, and median agreement on the embedded technical replicates must reach 100%; these consensus labels then serve as ground truth for the AI model trained on 2110 files.

## Related tools

- **iDIA-QC** (Standalone GUI platform whose machine learning models consume the expert-consensus labels produced by this agreement analysis as training ground truth) — github.com/guomics-lab/iDIA-QC

## Examples

```
python consensus_labels.py --annotations annotations_2638.csv --instruments instruments.tsv --metrics 15 --replicates 11 --out majority_vote_consensus_labels.csv --agreement observed_agreement_stats.tsv
```

## Evaluation signals

- Median observed agreement across the 21 instruments falls within the published 73–90% range
- Median observed agreement on the 11 embedded technical replicates reaches 100%
- Every one of the 2638 files receives exactly one consensus label per metric, aggregatable to a binary LC and MS status
- Consensus label distribution matches the 2110 training / 528 validation / 116 second-validation split used downstream
- Agreement computed on the same 15 metrics the experts were asked to assess, not the >140 candidate metrics

## Limitations

- Agreement is reported as observed agreement only — chance-corrected statistics (e.g. Cohen's/Fleiss' kappa) are not reported in this study
- Manual inspection covered 2754 DIA files with 21 experts, but the AI training set is a subset (2110 files), so consensus labels extend beyond what the model saw
- The 15-metric prioritization came from group effort narrowing >140 candidate metrics, so raters may not have assessed metrics outside this final set

## Evidence

- [methods] Agreement stats validation range: [evidence span withheld — anchor status: not_found]
