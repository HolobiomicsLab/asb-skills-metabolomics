---
name: lstm-sequence-detection-inference
description: Use when you have bacterial genome FASTA files and need domain-level
  BGC detection scores with reduced false positives, especially to detect novel BGC
  classes that HMM-based tools like ClusterFinder miss.
when_to_use_negative:
- Your inputs are already feature tables or Pfam TSVs without genome sequence — use
  deepbgc prepare / custom detectors rather than the full genome pipeline
- You need rule-based curation of known BGC types rather than machine-learning detection
  — antiSMASH-style approaches are more appropriate
- Your target organisms are poorly characterized non-Streptomyces bacteria and you
  need reliable novel-class assignment — the training data is heavily biased toward
  Streptomyces and most novel classes cannot be confidently assigned
tools:
- name: DeepBGC
  role: BiLSTM + pfam2vec detector executed via the deepbgc CLI to score domains and
    detect BGCs
  repo: github.com/Merck/deepbgc
- name: Prodigal
  role: Predicts protein-coding genes from the input genome FASTA before domain detection
  repo: github.com/hyattpd/Prodigal
- name: HMMER
  role: Detects Pfam protein domains used as the detector's tokenized input
- name: pfam2vec
  role: word2vec-like skip-gram embedding of Pfam domains feeding the BiLSTM
  repo: github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM baseline detector loaded via --detector for AUC comparison
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:aa6e35b16f80a666ac9159ace22311b1bb2f7ff7ccb2ccf91a4a224a584de88d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Merck/deepbgc
  tools:
  - ClusterFinder
  - DeepBGC
  - HMMER
  - Prodigal
  - pfam2vec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# lstm-sequence-detection-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect biosynthetic gene clusters (BGCs) in bacterial or fungal genomes by scoring Pfam domain sequences with a BiLSTM recurrent neural network over pfam2vec embeddings, as implemented in DeepBGC. The skill reduces false positive BGC detection relative to the ClusterFinder HMM (AUC 0.923 vs 0.847 on 65 validated BGCs).

## When to use

Use when you have bacterial genome FASTA files and need domain-level BGC detection scores with reduced false positives, especially to detect novel BGC classes that HMM-based tools like ClusterFinder miss.

## When NOT to use

- Your inputs are already feature tables or Pfam TSVs without genome sequence — use deepbgc prepare / custom detectors rather than the full genome pipeline
- You need rule-based curation of known BGC types rather than machine-learning detection — antiSMASH-style approaches are more appropriate
- Your target organisms are poorly characterized non-Streptomyces bacteria and you need reliable novel-class assignment — the training data is heavily biased toward Streptomyces and most novel classes cannot be confidently assigned

## Inputs

- Bacterial or fungal genome sequences in FASTA format
- DeepBGC trained detector models and Pfam database (via deepbgc download)
- pfam2vec embedding vectors (e.g. pfam2vec.csv from release v0.1.0)
- Custom ClusterFinder detector files (for comparison runs via --detector)
- Positive/negative labels per Pfam domain (in_cluster column) for validation

## Outputs

- Per-domain detection scores and per-BGC aggregate predictions with FASTA input coordinates
- Domain-level ROC curves and AUC ROC values (target: DeepBGC 0.923, ClusterFinder 0.847)
- Predicted BGC product class and activity classifications (random forest)
- antiSMASH-compatible JSON for visualization of predicted BGCs

## How to apply

Run 'deepbgc download' to fetch the trained detector models and the Pfam database, and confirm availability with 'deepbgc info'. Run 'deepbgc pipeline' on each genome FASTA; Prodigal predicts protein-coding genes and HMMER detects Pfam domains, which are converted to pfam2vec embedding vectors (trained from a large unlabeled corpus of genomic sequences) and fed to the Bidirectional LSTM detector to produce per-domain scores that are aggregated into BGCs. To benchmark against ClusterFinder, score the same Pfam domain sequences with the original ClusterFinder HMM and the retrained model loaded as custom detectors via --detector. Label each Pfam domain positive if it lies inside one of the 65 experimentally validated BGCs and negative if outside, then compute domain-level ROC curves and AUC from the per-domain prediction scores. Validate by reproducing DeepBGC AUC = 0.923 vs ClusterFinder AUC = 0.847. Detected BGCs can be classified by product class and activity using the supplemental random forest classifiers on Pfam domain composition.

## Related tools

- **DeepBGC** (BiLSTM + pfam2vec detector executed via the deepbgc CLI to score domains and detect BGCs) — github.com/Merck/deepbgc
- **Prodigal** (Predicts protein-coding genes from the input genome FASTA before domain detection) — github.com/hyattpd/Prodigal
- **HMMER** (Detects Pfam protein domains used as the detector's tokenized input)
- **pfam2vec** (word2vec-like skip-gram embedding of Pfam domains feeding the BiLSTM) — github.com/Merck/deepbgc
- **ClusterFinder** (HMM baseline detector loaded via --detector for AUC comparison) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc pipeline mySequence.fa --detector ClusterFinder.pfam.tsv
```

## Evaluation signals

- Reproduce domain-level AUC ROC = 0.923 for DeepBGC vs 0.847 for ClusterFinder on the 65 experimentally validated BGCs across six bacterial genomes
- 'deepbgc info' lists all downloaded dependencies and trained models before any inference is run
- Per-domain scores are present for all Pfam domains called by HMMER across the input genomes (positive labels inside validated BGCs, negatives in unannotated regions)
- Leave-class-out validation shows detection of BGC classes not encountered during training
- Random forest product-class predictions recover well-studied classes in the S. coelicolor reference genome example

## Limitations

- The model was trained on existing BGC databases heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting novel BGC detection in poorly characterized bacteria
- Some BGC classes were mis-identified in the S. coelicolor reference genome example, and recovery of some classes is threshold-dependent
- The majority of novel BGC classes identified could not be confidently assigned to a product class

## Evidence
