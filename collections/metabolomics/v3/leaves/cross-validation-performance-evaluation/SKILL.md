---
name: cross-validation-performance-evaluation
description: Use when comparing two or more BGC detectors on the same labeled Pfam
  training matrix and you need a fair, split-identical performance estimate such as
  mean ROC AUC across folds.
when_to_use_negative:
- You need detection on raw genomic sequence rather than detector benchmarking on
  pre-labeled Pfam TSV matrices — use the full deepbgc pipeline instead
- You want to evaluate class-level generalization to unseen BGC types — that requires
  leave-class-out validation, not standard 10-fold cross-validation
- The comparison target is a rule-based tool like antiSMASH, which does not consume
  the same Pfam-domain training splits
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0091
tools:
- name: DeepBGC
  role: BiLSTM RNN detector with pfam2vec embeddings, trained per fold via 'deepbgc
    train' and scored on held-out test splits
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: Hidden Markov Model baseline detector trained on the identical 10 train/test
    splits for AUC comparison
- name: pfam2vec
  role: word2vec-like Pfam domain embedding table loaded to encode training matrix
    rows for the DeepBGC detector
  repo: https://github.com/Merck/deepbgc
- name: bgc-pipeline
  role: DVC-managed repository holding cross-validation, Leave-Class-Out and Bootstrap
    evaluation code and data
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:a6d4747d37b44f1f35c048c33a72f37e9237657789c1eecd528b02f4eb7e6c11
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
  - bgc-pipeline
  - pfam2vec
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cross-validation-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark BGC detection models (e.g., DeepBGC BiLSTM RNN vs. ClusterFinder HMM) by scoring held-out test splits across 10-fold cross-validation on MIBiG positives vs. GeneSwap artificial negatives and comparing concatenated ROC AUC values.

## When to use

Use when comparing two or more BGC detectors on the same labeled Pfam training matrix and you need a fair, split-identical performance estimate such as mean ROC AUC across folds.

## When NOT to use

- You need detection on raw genomic sequence rather than detector benchmarking on pre-labeled Pfam TSV matrices — use the full deepbgc pipeline instead
- You want to evaluate class-level generalization to unseen BGC types — that requires leave-class-out validation, not standard 10-fold cross-validation
- The comparison target is a rule-based tool like antiSMASH, which does not consume the same Pfam-domain training splits

## Inputs

- Pfam TSV feature matrix for 1406 MIBiG positive BGCs with in_cluster label column
- GeneSwap_Negatives.pfam.tsv (10128 artificial negative clusters) from deepbgc release v0.1.0
- pfam2vec.csv embedding table (from deepbgc release v0.1.0)
- Detector JSON configuration for the KerasRNN DeepBGC detector
- ClusterFinder HMM detector

## Outputs

- Per-fold serialized detector models (one per fold per detector)
- Concatenated held-out test-split predictions from all 10 folds for each detector
- cv_roc_auc.csv containing per-fold and mean ROC AUC for DeepBGC and ClusterFinder

## How to apply

Assemble a labeled training matrix whose rows are Pfam TSV domain annotations: positive rows from 1406 MIBiG BGCs and negative rows from GeneSwap_Negatives.pfam.tsv (10128 negative clusters), each carrying a binary in_cluster label and a sequence_id that keeps each cluster's Pfam domains grouped so splits do not fragment a cluster. Load pfam2vec.csv embeddings and train the DeepBGC KerasRNN detector (hidden_size=128, batch_size=16, timesteps=256, learning_rate=0.0001, num_epochs=1000, early stopping on val_auc_roc with min_delta=0.0001 and patience=20, shuffle=True, weighted=True) with 'deepbgc train' on each of the 10 folds. Train the ClusterFinder HMM detector on the identical 10 train/test splits. Score each fold's held-out test split with both detectors, concatenate test-split predictions across folds, compute ROC AUC per fold and the mean for each model, and write per-fold and mean AUC values to cv_roc_auc.csv. The rationale is that identical splits and identical scoring isolate the detector as the only variable, replicating the paper's DeepBGC (AUC 0.984) vs. ClusterFinder (AUC 0.936) comparison.

## Related tools

- **DeepBGC** (BiLSTM RNN detector with pfam2vec embeddings, trained per fold via 'deepbgc train' and scored on held-out test splits) — https://github.com/Merck/deepbgc
- **ClusterFinder** (Hidden Markov Model baseline detector trained on the identical 10 train/test splits for AUC comparison)
- **pfam2vec** (word2vec-like Pfam domain embedding table loaded to encode training matrix rows for the DeepBGC detector) — https://github.com/Merck/deepbgc
- **bgc-pipeline** (DVC-managed repository holding cross-validation, Leave-Class-Out and Bootstrap evaluation code and data) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc train --matrix mibig_and_geneswap.pfam.tsv --detector myDetector --cv 10
```

## Evaluation signals

- cv_roc_auc.csv exists and contains an ROC AUC entry for both DeepBGC and ClusterFinder
- All reported ROC AUC values fall within [0, 1]
- The DeepBGC average ROC AUC exceeds the ClusterFinder average, consistent with the paper's reported 0.984 vs 0.936
- Each cluster's Pfam domains stay grouped under one sequence_id across folds (no cluster split between train and test)
- Per-fold AUC values are recorded individually, not only the mean, enabling fold-level variance inspection

## Limitations

- Training data are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, which can limit measured generalization
- Artificial GeneSwap negatives are not real non-BGC sequence, so the cross-validation AUC may overestimate discrimination on genuine genomes
- Further work is needed to curate more diverse BGC databases for improved training and validation

## Evidence

- [readme] evaluation data location: [evidence span withheld — anchor status: external_doc]
