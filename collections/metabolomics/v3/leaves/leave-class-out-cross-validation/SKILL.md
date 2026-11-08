---
name: leave-class-out-cross-validation
description: Use when evaluating whether a BGC detection model can extrapolate to
  unseen product classes, i.e. when the goal is leave-class-out validation across
  the six non-hybrid MIBiG classes rather than standard cross-validation on seen classes.
when_to_use_negative:
- The goal is ordinary performance on known classes — use standard cross-validation
  instead, since leave-class-out deliberately removes an entire class from training
- Input sequences have not been annotated to Pfam TSV format with an `in_cluster`
  column — prepare them first with `deepbgc prepare` and manual labeling
- Evaluating a rule-based tool like antiSMASH that does not train on class-labeled
  samples, so class holdout is not applicable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: DeepBGC
  role: Provides the KerasRNN detector training/scoreing framework, training data,
    negatives, and model templates used in the leave-class-out folds
  repo: https://github.com/Merck/deepbgc
- name: pfam2vec
  role: Word2vec-like skip-gram embedding of Pfam domains (pfam2vec.csv) used to encode
    gene features for the RNN detector
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM-based baseline detector whose leave-class-out AUC (0.865) is compared
    against DeepBGC's (0.946)
- name: bgc-pipeline
  role: Repository containing the development and evaluation code, including Leave-Class-Out
    evaluation data under data/evaluation
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:540e70b81a41a559c54cd03acb8ef0123f96f570465039e6dcd8938a60edffd7
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
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# leave-class-out-cross-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A generalizability test for the DeepBGC BiLSTM RNN detector in which, for each of the six non-hybrid MIBiG product classes, all samples of that class are held out during training and the trained detector is scored on the unseen class, quantifying extrapolation to BGC classes never encountered (AUC 0.946 vs 0.865 for ClusterFinder).

## When to use

Use when evaluating whether a BGC detection model can extrapolate to unseen product classes, i.e. when the goal is leave-class-out validation across the six non-hybrid MIBiG classes rather than standard cross-validation on seen classes.

## When NOT to use

- The goal is ordinary performance on known classes — use standard cross-validation instead, since leave-class-out deliberately removes an entire class from training
- Input sequences have not been annotated to Pfam TSV format with an `in_cluster` column — prepare them first with `deepbgc prepare` and manual labeling
- Evaluating a rule-based tool like antiSMASH that does not train on class-labeled samples, so class holdout is not applicable

## Inputs

- Positive BGC training data in Pfam TSV format (with `in_cluster` and `sequence_id` columns)
- GeneSwap_Negatives.pfam.tsv negative samples (https://github.com/Merck/deepbgc/releases/tag/v0.1.0)
- pfam2vec.csv embedding vectors (https://github.com/Merck/deepbgc/releases/tag/v0.1.0)
- JSON model templates from the DeepBGC v0.1.0 release

## Outputs

- leave_class_out_auc.csv with one mean AUC ROC entry per held-out MIBiG class (averaged over 3 repeats)
- Per-fold trained detector models saved as myDetector.pkl

## How to apply

Obtain the positive BGC training data in Pfam TSV format, the GeneSwap_Negatives.pfam.tsv negatives, and the pfam2vec.csv embedding vectors from the DeepBGC v0.1.0 release, plus the JSON model templates. For each of the six non-hybrid MIBiG classes, hold out all samples of that class and assemble a training set from the five remaining classes plus two-thirds of the negative samples, ensuring the required `in_cluster` (0 outside BGC, 1 inside) and `sequence_id` columns are present. Train a KerasRNN detector on pfam2vec-encoded features with hidden_size 128, timesteps 256, adam optimizer with learning_rate 0.0001, weighted classes, and early stopping monitoring val_auc_roc with patience 20 and min_delta 0.0001, saving each fold model as myDetector.pkl. Upsample the held-out class to 500 samples, score it with the trained detector, record the leave-class-out AUC ROC, and repeat the full train-test cycle three times per class; average the three repeats per class and write the six per-class mean AUC values to leave_class_out_auc.csv.

## Related tools

- **DeepBGC** (Provides the KerasRNN detector training/scoreing framework, training data, negatives, and model templates used in the leave-class-out folds) — https://github.com/Merck/deepbgc
- **pfam2vec** (Word2vec-like skip-gram embedding of Pfam domains (pfam2vec.csv) used to encode gene features for the RNN detector) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM-based baseline detector whose leave-class-out AUC (0.865) is compared against DeepBGC's (0.946))
- **bgc-pipeline** (Repository containing the development and evaluation code, including Leave-Class-Out evaluation data under data/evaluation) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc train --model myDetector.pkl --pipeline detection --positives mibig.pfam.tsv --negatives GeneSwap_Negatives.pfam.tsv --validation leave-class-out
```

## Evaluation signals

- leave_class_out_auc.csv exists and contains exactly one AUC entry per held-out class
- All AUC values fall within [0, 1]
- Per-class three-repeat averages reproduce the reported leave-class-out AUCs (overall DeepBGC AUC = 0.946 vs ClusterFinder = 0.865)
- Each fold model myDetector.pkl was trained without any samples of the held-out class (training set built only from the other five classes plus two-thirds of negatives)
- Held-out class test sets are upsampled to 500 samples before scoring

## Limitations

- Training data are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, so unseen-class extrapolation may not generalize to poorly characterized bacteria
- Further work is needed to curate more diverse BGC databases for improved training and validation of extrapolation
- Only the six non-hybrid MIBiG classes were evaluated; hybrid classes are not covered by this validation

## Evidence

- [readme] evaluation data location: [evidence span withheld — anchor status: external_doc]
