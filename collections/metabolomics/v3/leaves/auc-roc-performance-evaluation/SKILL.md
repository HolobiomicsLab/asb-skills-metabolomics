---
name: auc-roc-performance-evaluation
description: Use when comparing BGC detectors or any sequence-labeling classifier
  on held-out data and you need a threshold-independent performance metric (AUC ROC)
  to quantify detection accuracy, e.g., generalization to unseen classes.
when_to_use_negative:
- You need performance at a fixed decision threshold (e.g., genome-mining score cutoffs
  for BGC recovery, where the paper notes some classes are threshold-dependent) rather
  than a threshold-independent metric
- Your task is rule-based BGC boundary/annotation comparison (e.g., antiSMASH alignment),
  not probabilistic detection scoring
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: DeepBGC
  role: BiLSTM RNN detector trained per leave-class-out fold and scored to produce
    the AUC ROC values
  repo: https://github.com/Merck/deepbgc
- name: pfam2vec
  role: Provides the word2vec-like Pfam domain embeddings used to encode sequences
    for the scored detector
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM-based baseline detector whose AUC (0.865) is compared against DeepBGC's
    (0.946)
- name: bgc-pipeline
  role: 'Repository holding the leave-class-out evaluation code and data (data/evaluation:
    Cross-validation, Leave-Class-Out and Bootstrap evaluation)'
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:44fa401516fca548224a8b1f66448c6cd4c0cbcbdda5e16840fc81f3d4034a64
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
  - Keras
  - TensorFlow
  - bgc-pipeline
  - pfam2vec
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# auc-roc-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates binary BGC detection performance using AUC ROC, as in DeepBGC's leave-class-out validation where DeepBGC scored 0.946 versus ClusterFinder's 0.865 for identifying held-out, unseen BGC classes.

## When to use

Use when comparing BGC detectors or any sequence-labeling classifier on held-out data and you need a threshold-independent performance metric (AUC ROC) to quantify detection accuracy, e.g., generalization to unseen classes.

## When NOT to use

- You need performance at a fixed decision threshold (e.g., genome-mining score cutoffs for BGC recovery, where the paper notes some classes are threshold-dependent) rather than a threshold-independent metric
- Your task is rule-based BGC boundary/annotation comparison (e.g., antiSMASH alignment), not probabilistic detection scoring

## Inputs

- Positive BGC training data in Pfam TSV format with `in_cluster` and `sequence_id` columns (MIBiG-derived, six non-hybrid classes)
- GeneSwap_Negatives.pfam.tsv negative training samples (DeepBGC v0.1.0 release)
- pfam2vec.csv embedding vectors (DeepBGC v0.1.0 release)
- JSON model templates from the DeepBGC v0.1.0 release

## Outputs

- leave_class_out_auc.csv with one mean AUC ROC entry per held-out class (three-repeat average)
- Per-fold trained detector models saved as myDetector.pkl
- Comparative AUC summary (e.g., DeepBGC 0.946 vs ClusterFinder 0.865) demonstrating extrapolation to unseen BGC classes

## How to apply

Assemble positive BGC training samples in Pfam TSV format (with `in_cluster` and `sequence_id` columns), GeneSwap_Negatives.pfam.tsv negatives, and pfam2vec.csv embeddings from the DeepBGC v0.1.0 release. For each of the six non-hybrid MIBiG classes, hold out all samples of that class, train the KerasRNN detector on the remaining five classes plus two-thirds of negatives (hidden_size 128, timesteps 256, adam with learning_rate 0.0001, weighted classes, early stopping on val_auc_roc with patience 20 and min_delta 0.0001), and save each fold model as myDetector.pkl. Upsample the held-out class to 500 samples, score it with the fold detector, and record the AUC ROC, repeating the full train-test cycle three times per class. Average the three repeats per class, write the six per-class mean AUCs to leave_class_out_auc.csv, and compare detector-level averages (DeepBGC 0.946 vs ClusterFinder 0.865) to judge generalization to unseen BGC classes.

## Related tools

- **DeepBGC** (BiLSTM RNN detector trained per leave-class-out fold and scored to produce the AUC ROC values) — https://github.com/Merck/deepbgc
- **pfam2vec** (Provides the word2vec-like Pfam domain embeddings used to encode sequences for the scored detector) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM-based baseline detector whose AUC (0.865) is compared against DeepBGC's (0.946))
- **bgc-pipeline** (Repository holding the leave-class-out evaluation code and data (data/evaluation: Cross-validation, Leave-Class-Out and Bootstrap evaluation)) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc train --detector-config myDetector. --train-metadata data/mibig/train.pfam.tsv --validation-metadata data/mibig/validation.pfam.tsv --folds leave_class_out_auc.csv
```

## Evaluation signals

- leave_class_out_auc.csv exists and contains exactly one AUC entry per held-out class with all values in [0, 1]
- Per-class averages over the three train-test repeats reproduce the reported leave-class-out AUCs, with the detector mean near 0.946 for DeepBGC and 0.865 for ClusterFinder
- Each fold model (myDetector.pkl) was trained with early stopping monitoring val_auc_roc (patience 20, min_delta 0.0001), confirming the ROC metric was tracked during training
- Each held-out class was upsampled to 500 samples before scoring, so no class is scored with too few positives to estimate a stable AUC

## Limitations

- BGC training databases are heavily biased towards natural-product 'workhorses' such as Streptomyces, so AUC estimates may not transfer to poorly characterized taxa; more diverse databases are needed for improved training and validation
- BGC class recovery at fixed thresholds is threshold-dependent for some classes even when aggregate AUC is high, and some classes were mis-identified in the S. coelicolor reference genome example
- Six per-class AUC means come from only three repeats per class, so fold variance is not exhaustively characterized

## Evidence

- [results] Leave-class-out AUC comparison finding: [evidence span withheld — anchor status: not_found]
- [discussion] Leave-class-out rationale: "We addressed this extrapolation concern by performing leave-class-out validation to highlight the generalizability of our approach"
- [methods] Negative training data source: "You can use `GeneSwap_Negatives.pfam.tsv` from release https://github.com/Merck/deepbgc/releases/tag/v0.1.0"
- [methods] pfam2vec embeddings reuse: "you can reuse previously trained `pfam2vec.csv` results from https://github.com/Merck/deepbgc/releases/tag/v0.1.0"
- [methods] in_cluster labeling requirement: "you will need to manually add an `in_cluster` column that will contain 0 for pfams outside a BGC and 1 for pfams inside a BGC."
- [readme] Evaluation data location in repo: [evidence span withheld — anchor status: external_doc]
