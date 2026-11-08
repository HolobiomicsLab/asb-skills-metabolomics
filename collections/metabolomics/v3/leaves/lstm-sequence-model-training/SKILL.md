---
name: lstm-sequence-model-training
description: Use when training a BGC detection model on Pfam TSV gene features with
  pfam2vec embeddings, or when reproducing leave-class-out validation AUCs comparing
  DeepBGC against ClusterFinder.
when_to_use_negative:
- You only need to run detection with an already-trained model — use `deepbgc pipeline
  mySequence.fa` instead of training
- Your training examples are not in Pfam TSV format and have not been converted —
  run `deepbgc prepare` first, do not train on raw sequence
- The goal is rule-based BGC identification or profile-HMM comparison (e.g. antiSMASH-style),
  not deep-learning-based detection
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepBGC
  role: Deep learning framework whose `deepbgc train` command trains BiLSTM RNN detection/classification
    models on Pfam TSV inputs
  repo: https://github.com/Merck/deepbgc
- name: pfam2vec
  role: word2vec-like skip-gram embedding of Pfam domains used as RNN input features
  repo: https://github.com/Merck/deepbgc
- name: Merck/bgc-pipeline
  role: Development and evaluation code for leave-class-out, cross-validation, and
    bootstrap evaluation of the trained models
  repo: https://github.com/Merck/bgc-pipeline
- name: Prodigal
  role: Protein-coding gene prediction producing inputs for downstream Pfam annotation
    of training sequences
  repo: https://github.com/hyattpd/Prodigal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:e6fd6ff8f5bd4c32cc85bfc9c129e3d521cc0216b63a6abf1867aa35d32f045b
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
  - DeepBGC
  - Keras
  - Merck/bgc-pipeline
  - Prodigal
  - TensorFlow
  - pfam2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# lstm-sequence-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a Keras BiLSTM recurrent neural network detector on pfam2vec-encoded Pfam domain sequences to identify biosynthetic gene clusters, including leave-class-out validation to test extrapolation to BGC classes unseen during training.

## When to use

Use when training a BGC detection model on Pfam TSV gene features with pfam2vec embeddings, or when reproducing leave-class-out validation AUCs comparing DeepBGC against ClusterFinder.

## When NOT to use

- You only need to run detection with an already-trained model — use `deepbgc pipeline mySequence.fa` instead of training
- Your training examples are not in Pfam TSV format and have not been converted — run `deepbgc prepare` first, do not train on raw sequence
- The goal is rule-based BGC identification or profile-HMM comparison (e.g. antiSMASH-style), not deep-learning-based detection

## Inputs

- Positive BGC training data in Pfam TSV format with `in_cluster` and `sequence_id` columns
- GeneSwap_Negatives.pfam.tsv negative training samples (DeepBGC v0.1.0 release)
- pfam2vec.csv embedding vectors (DeepBGC v0.1.0 release)
- JSON model templates defining detector architecture and training parameters

## Outputs

- Trained detector model saved as myDetector.pkl
- leave_class_out_auc.csv with six per-class mean AUC ROC values
- Trained DeepBGC detection and classification models usable via `deepbgc pipeline`

## How to apply

Obtain the positive BGC training data in Pfam TSV format, the GeneSwap_Negatives.pfam.tsv negatives, and the pfam2vec.csv embedding vectors from the DeepBGC v0.1.0 release, plus the JSON model templates. Ensure the training TSVs contain the required `in_cluster` (0 for pfams outside a BGC, 1 inside) and `sequence_id` columns. Train a KerasRNN detector on pfam2vec-encoded features with hidden_size 128, 256 timesteps, adam optimizer at learning_rate 0.0001, weighted classes, and early stopping monitoring val_auc_roc with patience 20 and min_delta 0.0001, saving each fold model as myDetector.pkl. For leave-class-out validation, hold out one of the six non-hybrid MIBiG classes, train on the other five plus two-thirds of negatives, upsample the held-out class to 500 samples, and record AUC ROC over three repeats per class, averaging into leave_class_out_auc.csv. This design demonstrates that pfam2vec embeddings let the RNN generalize to unseen BGC classes (AUC 0.946 vs ClusterFinder's 0.865).

## Related tools

- **DeepBGC** (Deep learning framework whose `deepbgc train` command trains BiLSTM RNN detection/classification models on Pfam TSV inputs) — https://github.com/Merck/deepbgc
- **pfam2vec** (word2vec-like skip-gram embedding of Pfam domains used as RNN input features) — https://github.com/Merck/deepbgc
- **Merck/bgc-pipeline** (Development and evaluation code for leave-class-out, cross-validation, and bootstrap evaluation of the trained models) — https://github.com/Merck/bgc-pipeline
- **Prodigal** (Protein-coding gene prediction producing inputs for downstream Pfam annotation of training sequences) — https://github.com/hyattpd/Prodigal

## Examples

```
deepbgc train --help
```

## Evaluation signals

- leave_class_out_auc.csv exists and contains one AUC entry per held-out class with all values in [0, 1]
- Per-class mean AUCs over three train-test repeats reproduce the reported leave-class-out AUC of ~0.946 for DeepBGC versus ~0.865 for ClusterFinder
- Each fold model saved as myDetector.pkl trains until early stopping on val_auc_roc (patience 20, min_delta 0.0001) rather than exhausting all epochs
- Training TSVs contain the required `in_cluster` and `sequence_id` columns before training is launched
- Detector improves BGC identification accuracy within a genome relative to ClusterFinder, matching the paper's reported outperformance

## Limitations

- Trained models inherit bias from existing BGC databases that are heavily skewed towards Streptomyces 'workhorse' species, limiting novel-class detection in poorly characterized bacteria
- Further work is needed to curate more diverse BGC databases for improved training and validation
- In leave-class-out applications, the majority of novel BGC classes identified could not be confidently assigned, and some classes in the S. coelicolor reference genome were mis-identified
- BGC class recovery can be threshold-dependent: some well-studied classes were recovered regardless of threshold while others were not

## Evidence
