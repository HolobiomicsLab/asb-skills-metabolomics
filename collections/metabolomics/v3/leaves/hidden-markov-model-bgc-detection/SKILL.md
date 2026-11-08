---
name: hidden-markov-model-bgc-detection
description: Use when benchmarking HMM-based BGC detectors against alternative predictors
  on genomes with known validated BGCs, or when reproducing domain-level ROC/AUC comparisons
  such as ClusterFinder (AUC 0.847) vs DeepBGC (AUC 0.923).
when_to_use_negative:
- When the goal is rule-based product-class detection rather than positional accuracy
  benchmarking — antiSMASH-style rule-based approaches are a distinct comparison axis
- When input is already a Pfam TSV feature table with no genome sequence or validated
  BGC ground truth, no AUC evaluation is possible
- When seeking detection of novel BGC classes beyond training bias — note the training
  databases were heavily biased towards Streptomyces 'workhorse' BGCs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ClusterFinder
  role: Original HMM-based machine learning BGC detector used as the baseline comparator
    scored on the same Pfam domain sequences
- name: DeepBGC
  role: Deep learning detector (BiLSTM RNN over pfam2vec embeddings) evaluated head-to-head
    against the ClusterFinder HMM via domain-level AUC; used with --detector for custom
    models
  repo: https://github.com/Merck/deepbgc
- name: HMMER
  role: Detects Pfam domains in predicted proteins to produce the domain sequence
    scored by the HMMs
- name: Prodigal
  role: Predicts protein-coding genes from the input genome FASTA before Pfam domain
    detection
  repo: https://github.com/hyattpd/Prodigal/releases
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:16ec9d9610cc0a711ac97c09bb72fea7669a8497bd50e6200974be99db9b4f83
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Merck/deepbgc
  repo_url: https://github.com/Merck/deepbgc
  tools:
  - ClusterFinder
  - DeepBGC
  - HMMER
  - Prodigal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hidden-markov-model-bgc-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects biosynthetic gene clusters (BGCs) in bacterial genomes by scoring Pfam protein-domain sequences with Hidden Markov Models (e.g., the original ClusterFinder HMM) and comparing detection performance against deep-learning detectors such as DeepBGC at the domain level via ROC/AUC.

## When to use

Use when benchmarking HMM-based BGC detectors against alternative predictors on genomes with known validated BGCs, or when reproducing domain-level ROC/AUC comparisons such as ClusterFinder (AUC 0.847) vs DeepBGC (AUC 0.923).

## When NOT to use

- When the goal is rule-based product-class detection rather than positional accuracy benchmarking — antiSMASH-style rule-based approaches are a distinct comparison axis
- When input is already a Pfam TSV feature table with no genome sequence or validated BGC ground truth, no AUC evaluation is possible
- When seeking detection of novel BGC classes beyond training bias — note the training databases were heavily biased towards Streptomyces 'workhorse' BGCs

## Inputs

- Bacterial genome FASTA files (e.g., six genomes containing 65 experimentally validated BGCs)
- Trained detector model files (original ClusterFinder HMM, retrained ClusterFinder model, DeepBGC detector)
- Pfam database
- Ground-truth annotation of validated BGC coordinates for labeling Pfam domains as in_cluster (1) or outside (0)

## Outputs

- Per-domain prediction scores for each detector
- Domain-level ROC curves and AUC ROC values per tool (e.g., DeepBGC 0.923, ClusterFinder 0.847)
- Predicted BGC regions and cluster annotations per genome

## How to apply

Download the trained detector models and Pfam database with 'deepbgc download' and verify dependencies with 'deepbgc info'. Run the pipeline on each genome FASTA file, letting Prodigal call proteins and HMMER detect Pfam domains automatically. Score the same genomes' Pfam domain sequences with the original ClusterFinder HMM (and optionally the retrained ClusterFinder model) loaded as custom detector files via --detector. Label each Pfam domain positive if it lies within an experimentally validated BGC (e.g., the 65 validated BGCs across six bacterial genomes) and negative if in an unannotated region, then compute domain-level ROC curves and AUC ROC from per-domain prediction scores to compare tools.

## Related tools

- **ClusterFinder** (Original HMM-based machine learning BGC detector used as the baseline comparator scored on the same Pfam domain sequences)
- **DeepBGC** (Deep learning detector (BiLSTM RNN over pfam2vec embeddings) evaluated head-to-head against the ClusterFinder HMM via domain-level AUC; used with --detector for custom models) — https://github.com/Merck/deepbgc
- **HMMER** (Detects Pfam domains in predicted proteins to produce the domain sequence scored by the HMMs)
- **Prodigal** (Predicts protein-coding genes from the input genome FASTA before Pfam domain detection) — https://github.com/hyattpd/Prodigal/releases

## Examples

```
deepbgc download && deepbgc pipeline --detector clusterfinder.hmm mySequence.fa
```

## Evaluation signals

- Computed domain-level AUC ROC for the original ClusterFinder HMM on the 65 validated BGC set reproduces the published value of 0.847
- Computed domain-level AUC ROC for DeepBGC reproduces 0.923, with DeepBGC outperforming both ClusterFinder variants
- 'deepbgc info' confirms the trained models and Pfam database downloaded successfully before scoring
- Every Pfam domain receives a binary in_cluster label (0 outside BGC, 1 inside) before ROC computation
- The same per-domain score table structure is produced for each detector, enabling direct ROC curve overlay

## Limitations

- Training data bias: existing BGC databases are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting novel BGC detection in poorly characterized bacteria
- Some well-studied BGC classes were recovered regardless of threshold while others were more threshold-dependent, complicating class-level comparisons
- Some BGC classes were mis-identified in the S. coelicolor reference genome example

## Evidence
