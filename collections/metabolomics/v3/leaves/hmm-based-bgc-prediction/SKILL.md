---
name: hmm-based-bgc-prediction
description: Use when benchmarking a deep-learning BGC detector against a probabilistic
  HMM baseline, or when training ClusterFinder on Pfam TSV training data with in_cluster
  labels.
when_to_use_negative:
- The goal is production BGC detection in new genomes — the HMM is the weaker comparator
  and the paper shows DeepBGC outperforms it in both accuracy and novel-class detection
- Input is raw nucleotide sequence rather than Pfam-annotated domain TSVs; sequences
  must first be prepared (e.g. deepbgc prepare) into Pfam TSV format
- Evaluating extrapolation to BGC classes absent from training data — that requires
  the leave-class-out analysis, not this cross-validation setup
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0154
tools:
- name: ClusterFinder
  role: Hidden Markov Model BGC detector trained on the same 10-fold splits as the
    deep-learning detector to provide the baseline ROC AUC
- name: DeepBGC
  role: Deep-learning comparator whose BiLSTM RNN detector is benchmarked head-to-head
    against the HMM baseline in cross-validation
  repo: https://github.com/Merck/deepbgc
- name: pfam2vec
  role: word2vec-like embedding of Pfam domains used by the deep-learning comparator
    on the same Pfam-encoded inputs
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:b5ae26993435b32a8f9367cb63c0939a22cf26bc6ddd70c8fc5e0fbf56e62f63
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
  - TensorFlow
  - pfam2vec
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hmm-based-bgc-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains and scores a Hidden Markov Model BGC detector (ClusterFinder) on Pfam-domain training matrices as the classical machine-learning baseline against which DeepBGC's BiLSTM RNN detector is compared in 10-fold cross-validation on MIBiG positives vs artificial negatives.

## When to use

Use when benchmarking a deep-learning BGC detector against a probabilistic HMM baseline, or when training ClusterFinder on Pfam TSV training data with in_cluster labels.

## When NOT to use

- The goal is production BGC detection in new genomes — the HMM is the weaker comparator and the paper shows DeepBGC outperforms it in both accuracy and novel-class detection
- Input is raw nucleotide sequence rather than Pfam-annotated domain TSVs; sequences must first be prepared (e.g. deepbgc prepare) into Pfam TSV format
- Evaluating extrapolation to BGC classes absent from training data — that requires the leave-class-out analysis, not this cross-validation setup

## Inputs

- Positive Pfam TSVs for 1406 MIBiG BGCs (with in_cluster=1 labels)
- GeneSwap_Negatives.pfam.tsv artificial negative clusters (10128 clusters, in_cluster=0)
- Pfam training TSV with in_cluster column and sequence_id grouping
- Shared 10-fold train/test split definitions

## Outputs

- Trained ClusterFinder HMM detector model per fold
- Test-split BGC predictions/scores per fold
- cv_roc_auc.csv with per-fold and mean ROC AUC for ClusterFinder (expected mean ≈ 0.936)

## How to apply

Assemble a Pfam TSV training matrix containing positive Pfam domain sequences for the 1406 MIBiG BGCs and GeneSwap_Negatives.pfam.tsv (10128 artificial negative clusters), each row carrying a binary in_cluster label and a sequence_id that keeps each cluster's Pfam domains grouped. Train the ClusterFinder Hidden Markov Model detector on exactly the same 10 train/test splits used by the DeepBGC Keras RNN detector (hidden_size=128, batch_size=16, timesteps=256, learning_rate=0.0001) so the comparison is paired. Score each fold's held-out test split with the trained HMM and concatenate predictions across the 10 folds. Compute ROC AUC per fold and the mean, writing per-fold and mean values to cv_roc_auc.csv for direct comparison against DeepBGC (paper reference: 0.936 HMM vs 0.984 DeepBGC).

## Related tools

- **ClusterFinder** (Hidden Markov Model BGC detector trained on the same 10-fold splits as the deep-learning detector to provide the baseline ROC AUC)
- **DeepBGC** (Deep-learning comparator whose BiLSTM RNN detector is benchmarked head-to-head against the HMM baseline in cross-validation) — https://github.com/Merck/deepbgc
- **pfam2vec** (word2vec-like embedding of Pfam domains used by the deep-learning comparator on the same Pfam-encoded inputs) — https://github.com/Merck/deepbgc

## Examples

```
deepbgc train --detector clusterfinder_hmm mibig_positives.pfam.tsv GeneSwap_Negatives.pfam.tsv
```

## Evaluation signals

- cv_roc_auc.csv exists and contains an ROC AUC entry for both ClusterFinder and DeepBGC with values in [0,1]
- ClusterFinder mean ROC AUC across 10 folds approximates the reported 0.936 (well below DeepBGC's 0.984)
- All 10 folds use identical train/test splits for both detectors so AUC values are paired and comparable
- Concatenated test-split predictions cover every sample exactly once across the 10 folds
- Per-fold AUC values are finite and non-degenerate (not 0.5, indicating the HMM actually learned the MIBiG-vs-negative boundary)

## Limitations

- Artificial random negative clusters are not real non-BGC genomic regions, so the cross-validation AUC may overstate discrimination performance on real genomes
- ClusterFinder's HMM baseline is outperformed by DeepBGC here; using it as the final detector forfeits reduced false positive rates and novel-class detection
- Training data biases towards Streptomyces 'workhorse' BGCs affect the HMM baseline as well as the deep model

## Evidence

- [results] DeepBGC outperformed ClusterFinder at discriminating positive from negative samples, achieving AUC 0.984 versus 0.936.: [evidence span withheld — anchor status: not_found]
- [readme] This repository provides data and examples that were used for development of DeepBGC and its evaluation with ClusterFinder and antiSMASH.: [evidence span withheld — anchor status: external_doc]
