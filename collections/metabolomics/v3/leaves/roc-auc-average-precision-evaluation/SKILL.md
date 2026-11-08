---
name: roc-auc-average-precision-evaluation
description: Use when comparing gene-cluster detection methods (e.g., a trained deep-learning
  detector vs. ClusterFinder HMMs) on genomes with known BGC annotations and you need
  area-under-ROC-curve and average-precision metrics over bootstrap resamples.
when_to_use_negative:
- The task is BGC product-class or activity classification — this skill only evaluates
  detection (in-cluster vs out-of-cluster) performance, not the downstream random
  forest classifiers for product class and chemical activity.
- Input is raw genomic sequence without annotated BGC/non-BGC regions — ROC AUC and
  average precision cannot be computed without ground-truth labels on the nine-genome
  validation set.
- You need rule-based (antiSMASH-style) BGC detection rather than supervised detection-model
  benchmarking against ClusterFinder HMMs.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0797
tools:
- name: DeepBGC
  role: BiLSTM detector being benchmarked; trained with `deepbgc prepare` and `deepbgc
    train` using pfam2vec embeddings and evaluated via `deepbgc pipeline`
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: Baseline HMM detection method (original and retrained variants) compared against
    DeepBGC in the bootstrap evaluation
- name: pfam2vec
  role: word2vec-like Pfam domain embedding vectors (pfam2vec.csv) supplied as input
    features to the detector via --config
  repo: https://github.com/Merck/deepbgc
- name: Prodigal
  role: Protein-coding gene prediction used to produce Pfam-domain input features
    for BGC detection
  repo: https://github.com/hyattpd/Prodigal
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:8a85abc007b3c88fed4faf147e76597df819f5de8d49b828c3f546dd9e95d3bf
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
  - Prodigal
  - pfam2vec
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# roc-auc-average-precision-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark BGC detection performance of DeepBGC's BiLSTM detector against ClusterFinder HMMs using a 5-iteration bootstrap evaluation over nine manually BGC-annotated whole genomes, reporting ROC AUC and average precision per method.

## When to use

Use when comparing gene-cluster detection methods (e.g., a trained deep-learning detector vs. ClusterFinder HMMs) on genomes with known BGC annotations and you need area-under-ROC-curve and average-precision metrics over bootstrap resamples.

## When NOT to use

- The task is BGC product-class or activity classification — this skill only evaluates detection (in-cluster vs out-of-cluster) performance, not the downstream random forest classifiers for product class and chemical activity.
- Input is raw genomic sequence without annotated BGC/non-BGC regions — ROC AUC and average precision cannot be computed without ground-truth labels on the nine-genome validation set.
- You need rule-based (antiSMASH-style) BGC detection rather than supervised detection-model benchmarking against ClusterFinder HMMs.

## Inputs

- Validation contigs with annotated BGC and non-BGC regions (deepbgc release v0.1.5)
- GeneSwap_Negatives.pfam.tsv negative set (deepbgc release v0.1.0)
- pfam2vec.csv embedding vectors (deepbgc release v0.1.0)
- Trained BiLSTM detector model (KerasRNN JSON config: hidden_size 128, timesteps 256, adam lr 0.0001, stateful, weighted, early stopping on val_auc_roc patience 20 min_delta 0.0001)
- Pfam TSV training examples with manually added in_cluster (0/1) and sequence_id columns
- Nine manually BGC-annotated whole genomes

## Outputs

- Per-method ROC AUC and average precision over 5 bootstrap iterations
- detection_benchmark_metrics.csv aggregating averaged per-method metrics
- Validation AUC ROC learning curve showing monotonic improvement until early stopping

## How to apply

Prepare positive and negative training examples in Pfam TSV format via `deepbgc prepare`, then manually add an `in_cluster` column (1 for Pfams inside a BGC, 0 outside) and a `sequence_id` column so each sample's Pfams stay grouped during shuffling. Fill the KerasRNN detector JSON template with the documented settings (hidden_size 128, timesteps 256, adam optimizer, learning rate 0.0001, num_epochs up to 1000, stateful true, weighted true, early stopping on val_auc_roc with patience 20 and min_delta 0.0001), using Pfam2VecTransformer plus ProteinBorderTransformer input features, then train the BiLSTM detector with `deepbgc train` on the 617-positive/10128-negative ClusterFinder-derived set, supplying previously trained pfam2vec.csv vectors via --config. Download validation contigs with annotated BGC and non-BGC regions from the deepbgc releases (v0.1.5) and GeneSwap_Negatives.pfam.tsv plus pfam2vec.csv from v0.1.0. Evaluate the trained detector alongside the original and retrained ClusterFinder HMMs over a 5-iteration bootstrap split of the nine BGC-annotated genomes, computing ROC AUC and average precision per method. Aggregate the averaged per-method metrics into detection_benchmark_metrics.csv and compare against the published reference values (DeepBGC AUC 0.946 / AP 0.75; original ClusterFinder 0.837 / 0.28; retrained ClusterFinder 0.912 / 0.63).

## Related tools

- **DeepBGC** (BiLSTM detector being benchmarked; trained with `deepbgc prepare` and `deepbgc train` using pfam2vec embeddings and evaluated via `deepbgc pipeline`) — https://github.com/Merck/deepbgc
- **ClusterFinder** (Baseline HMM detection method (original and retrained variants) compared against DeepBGC in the bootstrap evaluation)
- **pfam2vec** (word2vec-like Pfam domain embedding vectors (pfam2vec.csv) supplied as input features to the detector via --config) — https://github.com/Merck/deepbgc
- **Prodigal** (Protein-coding gene prediction used to produce Pfam-domain input features for BGC detection) — https://github.com/hyattpd/Prodigal

## Examples

```
deepbgc train --config keras_rnn_detector. --pfam2vec pfam2vec.csv --out deepbgc_detector
```

## Evaluation signals

- Reproduced metrics match published values: DeepBGC ROC AUC 0.946 / average precision 0.75; original ClusterFinder 0.837 / 0.28; retrained ClusterFinder 0.912 / 0.63.
- Trained detector's validation AUC ROC improves monotonically until early stopping is triggered (patience 20, min_delta 0.0001 on val_auc_roc).
- Training set composition matches the expected 617-positive / 10128-negative ClusterFinder-derived samples after `deepbgc prepare` with manually added in_cluster and sequence_id columns.
- detection_benchmark_metrics.csv contains averaged per-method metrics across all 5 bootstrap iterations of the nine-genome validation.
- pfam2vec.csv vectors and GeneSwap_Negatives.pfam.tsv negative set loaded from the correct deepbgc release tags (v0.1.0).

## Limitations

- Training data bias: BGC databases are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting novel BGC detection in poorly characterized bacteria.
- The evaluation is limited to nine manually BGC-annotated whole genomes with a 5-iteration bootstrap; more diverse BGC databases are needed to improve training and validation.
- Only detection performance (in-cluster vs out-of-cluster) is benchmarked — downstream product-class and activity classification by random forest classifiers is evaluated separately.

## Evidence

- [results] bootstrap nine-genome validation AUC comparison: [evidence span withheld — anchor status: not_found]
- [methods] detector training data size: [evidence span withheld — anchor status: not_found]
