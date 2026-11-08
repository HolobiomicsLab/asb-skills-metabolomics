---
name: bootstrap-resampling-benchmarking
description: Use when comparing BGC detection performance of multiple trained detectors
  (e.g., a BiLSTM DeepBGC detector vs ClusterFinder HMMs) on manually annotated whole
  genomes and you need resampled, averaged AUC/precision estimates.
when_to_use_negative:
- Task is BGC product-class or activity prediction (use the random forest classifiers)
  rather than detection performance benchmarking
- Genomes lack manual BGC annotations, so no ground truth exists for computing ROC
  AUC / average precision
- Evaluating rule-based tools such as antiSMASH, which the paper treats as a separate
  rule-based comparison rather than this bootstrap detection benchmark
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: DeepBGC
  role: Provides deepbgc prepare/train commands and release assets (pfam2vec.csv,
    GeneSwap_Negatives.pfam.tsv, validation contigs) used to train and evaluate the
    BiLSTM detector in the bootstrap benchmark
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM-based baseline detector (original and retrained variants) benchmarked
    against DeepBGC over the bootstrap split
- name: pfam2vec
  role: Word2vec-like Pfam domain embeddings supplied to the BiLSTM detector as input
    features for the evaluated model
- name: bgc-pipeline
  role: Repository holding data and evaluation code (bootstrap, cross-validation,
    leave-class-out) used to reproduce the benchmark via DVC
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:0cd9147dcec751cc84c02eb4da5536f1c29f3f0f877f98229fd9e47c0f50c26f
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
  - bgc-pipeline
  - pfam2vec
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bootstrap-resampling-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark BGC-detection methods (DeepBGC vs original and retrained ClusterFinder HMMs) by evaluating ROC AUC and average precision over a 5-iteration bootstrap split of nine manually BGC-annotated whole genomes, producing averaged per-method detection metrics.

## When to use

Use when comparing BGC detection performance of multiple trained detectors (e.g., a BiLSTM DeepBGC detector vs ClusterFinder HMMs) on manually annotated whole genomes and you need resampled, averaged AUC/precision estimates.

## When NOT to use

- Task is BGC product-class or activity prediction (use the random forest classifiers) rather than detection performance benchmarking
- Genomes lack manual BGC annotations, so no ground truth exists for computing ROC AUC / average precision
- Evaluating rule-based tools such as antiSMASH, which the paper treats as a separate rule-based comparison rather than this bootstrap detection benchmark

## Inputs

- Validation contigs from nine manually BGC-annotated whole genomes (DeepBGC release 0.1.5)
- Trained DeepBGC BiLSTM detector (KerasRNN JSON config with pfam2vec.csv vectors)
- Original ClusterFinder HMM
- ClusterFinder HMM retrained with up-to-date data
- Pfam TSV training examples with in_cluster and sequence_id columns (617 positive / 10128 negative)

## Outputs

- Per-method ROC AUC and average precision per bootstrap iteration
- Aggregated averaged metrics table (detection_benchmark_metrics.csv)

## How to apply

Download validation contigs with annotated BGC and non-BGC regions from the DeepBGC release 0.1.5, plus GeneSwap_Negatives.pfam.tsv and pfam2vec.csv from release 0.1.0. Train the BiLSTM detector with deepbgc train on the 617-positive/10128-negative ClusterFinder-derived set, using a KerasRNN config with hidden_size 128, timesteps 256, adam optimizer at learning rate 0.0001, up to 1000 epochs, stateful/weighted true, and early stopping on val_auc_roc (patience 20, min_delta 0.0001) with Pfam2VecTransformer plus ProteinBorderTransformer input features. Evaluate the trained detector alongside the original and retrained ClusterFinder HMMs over a 5-iteration bootstrap split of the nine BGC-annotated genomes, computing ROC AUC and average precision per method per iteration. Aggregate the averaged per-method metrics into detection_benchmark_metrics.csv and compare against the reported values to confirm correct reproduction.

## Related tools

- **DeepBGC** (Provides deepbgc prepare/train commands and release assets (pfam2vec.csv, GeneSwap_Negatives.pfam.tsv, validation contigs) used to train and evaluate the BiLSTM detector in the bootstrap benchmark) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM-based baseline detector (original and retrained variants) benchmarked against DeepBGC over the bootstrap split)
- **pfam2vec** (Word2vec-like Pfam domain embeddings supplied to the BiLSTM detector as input features for the evaluated model)
- **bgc-pipeline** (Repository holding data and evaluation code (bootstrap, cross-validation, leave-class-out) used to reproduce the benchmark via DVC) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc train pfam2vec.csv config/kerasrnn. --labels in_cluster training_pfam.tsv
```

## Evaluation signals

- Averaged DeepBGC ROC AUC ≈ 0.946 and average precision ≈ 0.75 on the nine-genome bootstrap evaluation
- Original ClusterFinder HMM AUC ≈ 0.837 / average precision ≈ 0.28; retrained ClusterFinder AUC ≈ 0.912 / average precision ≈ 0.63
- DeepBGC should outperform both ClusterFinder variants on both metrics in the bootstrap evaluation
- Trained detector's validation AUC ROC improves monotonically until early stopping triggers (patience 20, min_delta 0.0001)
- Aggregated output matches detection_benchmark_metrics.csv schema with averaged per-method metrics over 5 bootstrap iterations

## Limitations

- Training data bias: BGC databases are heavily biased towards natural-product 'workhorses' such as Streptomyces, limiting generalization
- Some BGC classes in validation genomes may be mis-identified or threshold-dependent, affecting ground-truth metrics
- Reproduction requires exact release assets (0.1.0 and 0.1.5) and DVC-managed data from the bgc-pipeline repository

## Evidence

- [readme] Bootstrap evaluation data is managed in the bgc-pipeline repository: [evidence span withheld — anchor status: external_doc]
