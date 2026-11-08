---
name: class-imbalance-upsampling
description: Use when training or evaluating a BGC detector on class-imbalanced Pfam
  TSV training sets — e.g., scoring a held-out MIBiG class with too few examples for
  stable AUC estimation, or when ClusterFinder-comparable leave-class-out AUCs are
  the target metric.
when_to_use_negative:
- Input is already balanced or the goal is standard full-class cross-validation where
  all six classes appear in both train and test — leave-class-out upsampling would
  leak or distort class composition
- The task is product-class or molecular-activity classification with the supplemental
  random forest classifiers, not BiLSTM detection scoring of an unseen class
- Input is raw genome FASTA needing Prodigal annotation and `deepbgc prepare` rather
  than pre-computed Pfam TSV features
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepBGC
  role: Provides the KerasRNN detector (with class weighting) and the `deepbgc train`
    interface used to fit per-fold models on the imbalanced Pfam TSV data
  repo: https://github.com/Merck/deepbgc
- name: pfam2vec
  role: Supplies the word2vec-like Pfam domain embeddings used to encode upsampled
    and weighted training samples for the RNN
  repo: https://github.com/Merck/deepbgc
- name: bgc-pipeline
  role: Contains the Leave-Class-Out evaluation code and DVC-managed data files used
    to orchestrate the upsampling and AUC scoring cycles
  repo: https://github.com/Merck/bgc-pipeline
- name: ClusterFinder
  role: Baseline HMM detector whose leave-class-out AUC (0.865) is the comparison
    target for the upsampled DeepBGC evaluation (0.946)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:0cec17540543a3ff77b90fdffe493fb4c06bfcd53cdabcf85214e4d0435d0d0f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# class-imbalance-upsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compensates for severe class imbalance when training and validating DeepBGC's BiLSTM detector on Pfam TSV data by (a) passing weighted classes to the KerasRNN detector during training and (b) upsampling a rare/held-out MIBiG class to 500 samples before scoring, so that leave-class-out AUC ROC is measurable for small classes.

## When to use

Use when training or evaluating a BGC detector on class-imbalanced Pfam TSV training sets — e.g., scoring a held-out MIBiG class with too few examples for stable AUC estimation, or when ClusterFinder-comparable leave-class-out AUCs are the target metric.

## When NOT to use

- Input is already balanced or the goal is standard full-class cross-validation where all six classes appear in both train and test — leave-class-out upsampling would leak or distort class composition
- The task is product-class or molecular-activity classification with the supplemental random forest classifiers, not BiLSTM detection scoring of an unseen class
- Input is raw genome FASTA needing Prodigal annotation and `deepbgc prepare` rather than pre-computed Pfam TSV features

## Inputs

- Positive BGC training data in Pfam TSV format with `in_cluster` and `sequence_id` columns (from MIBiG samples)
- GeneSwap_Negatives.pfam.tsv negatives (from DeepBGC v0.1.0 release)
- pfam2vec.csv embedding vectors (from DeepBGC v0.1.0 release)
- JSON model templates for the KerasRNN detector

## Outputs

- Per-fold trained detector saved as myDetector.pkl
- Upsampled held-out class scores (500 samples per fold) with leave-class-out AUC ROC values
- leave_class_out_auc.csv with six per-class mean AUC values (three-repeat averages)

## How to apply

Assemble per-fold training sets from the five retained non-hybrid MIBiG classes plus two-thirds of GeneSwap_Negatives.pfam.tsv, keeping the required `in_cluster` and `sequence_id` columns, then train the KerasRNN detector (hidden_size 128, timesteps 256, adam at learning_rate 0.0001, early stopping on val_auc_roc with patience 20 and min_delta 0.0001) with class weighting enabled so abundant classes do not dominate the loss. For each held-out class, upsample its samples to 500 before scoring with the fold model, because raw class sizes are far too small for a reliable AUC ROC estimate. Repeat the full train-test cycle three times per class and average the AUCs; the workflow writes the six per-class mean AUCs to leave_class_out_auc.csv. The rationale is that DeepBGC's reported ability to extrapolate to unseen classes (AUC 0.946 vs ClusterFinder's 0.865) can only be reproduced if each rare held-out class contributes enough scored positives per fold.

## Related tools

- **DeepBGC** (Provides the KerasRNN detector (with class weighting) and the `deepbgc train` interface used to fit per-fold models on the imbalanced Pfam TSV data) — https://github.com/Merck/deepbgc
- **pfam2vec** (Supplies the word2vec-like Pfam domain embeddings used to encode upsampled and weighted training samples for the RNN) — https://github.com/Merck/deepbgc
- **bgc-pipeline** (Contains the Leave-Class-Out evaluation code and DVC-managed data files used to orchestrate the upsampling and AUC scoring cycles) — https://github.com/Merck/bgc-pipeline
- **ClusterFinder** (Baseline HMM detector whose leave-class-out AUC (0.865) is the comparison target for the upsampled DeepBGC evaluation (0.946))

## Examples

```
deepbgc train myDetector my_training.pfam.tsv
```

## Evaluation signals

- Each held-out class is upsampled to exactly 500 samples before scoring; check scored positive counts per fold equal 500
- leave_class_out_auc.csv exists and contains exactly one AUC entry per held-out class with values in [0, 1]
- Per-class averages across the three train-test repeats should aggregate toward the reported leave-class-out AUC of 0.946 for DeepBGC vs 0.865 for ClusterFinder
- Training set composition per fold: five classes plus two-thirds of negatives, with `in_cluster` (0/1) and `sequence_id` columns present
- Detector config confirms class weighting and early stopping on val_auc_roc (patience 20, min_delta 0.0001) were active

## Limitations

- BGC training databases are heavily biased towards 'workhorse' organisms such as Streptomyces, so class imbalance partly reflects database bias and limits novel-class detection in poorly characterized bacteria
- In genome applications the majority of newly detected BGC classes could not be confidently assigned, and some class recovery was threshold-dependent
- Some well-studied classes were recovered regardless of threshold while others were threshold-dependent, so upsampled AUC gains do not guarantee robust class identification at any single score threshold

## Evidence

- [readme] Evaluation data organization includes Leave-Class-Out: [evidence span withheld — anchor status: external_doc]
