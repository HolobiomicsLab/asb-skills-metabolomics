---
name: lstm-detector-training
description: Use when training a deep learning BGC detector from labeled Pfam TSV
  training examples (MIBiG positives vs. artificial negatives) and you need cross-validated
  AUC comparison to an HMM baseline like ClusterFinder.
when_to_use_negative:
- Your goal is rule-based BGC identification in a genome rather than training a detector
  from labeled data — use antiSMASH or deepbgc pipeline instead
- You lack the pfam2vec.csv embeddings or labeled Pfam TSV training data (in_cluster
  column) — the detector cannot be trained without them
- You only need product class or activity classification of already-detected BGCs
  — use the random forest classifiers, not the RNN detector
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0154
tools:
- name: DeepBGC
  role: BiLSTM RNN detector trained via deepbgc train on Pfam TSV examples with pfam2vec
    embeddings
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: Hidden Markov Model baseline detector trained on identical 10-fold splits
    for AUC comparison
- name: pfam2vec
  role: word2vec-like skip-gram embedding of Pfam domains used as detector input features
    (pfam2vec.csv)
  repo: https://github.com/Merck/deepbgc
- name: bgc-pipeline
  role: DVC-managed code and data for DeepBGC development and ClusterFinder/antiSMASH
    evaluation
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:2ba8b108d42bb57e36ec9a21a5024f49ef5c60a6a653b5c45b3f93121ff42618
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# lstm-detector-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a BiLSTM recurrent neural network BGC detector (DeepBGC) on Pfam domain sequences using word2vec-like pfam2vec embeddings, with binary in_cluster labels, and evaluate it against ClusterFinder via 10-fold cross-validated ROC AUC.

## When to use

Use when training a deep learning BGC detector from labeled Pfam TSV training examples (MIBiG positives vs. artificial negatives) and you need cross-validated AUC comparison to an HMM baseline like ClusterFinder.

## When NOT to use

- Your goal is rule-based BGC identification in a genome rather than training a detector from labeled data — use antiSMASH or deepbgc pipeline instead
- You lack the pfam2vec.csv embeddings or labeled Pfam TSV training data (in_cluster column) — the detector cannot be trained without them
- You only need product class or activity classification of already-detected BGCs — use the random forest classifiers, not the RNN detector

## Inputs

- Positive Pfam TSV training files for 1406 MIBiG BGCs
- GeneSwap_Negatives.pfam.tsv (10128 artificial negative clusters) from deepbgc release v0.1.0
- pfam2vec.csv embedding table from deepbgc release v0.1.0
- Detector configuration JSON template (hidden_size, batch_size, timesteps, learning_rate, epochs, early stopping settings)
- in_cluster binary labels and sequence_id columns in the training TSV

## Outputs

- 10-fold cross-validated serialized detector models (myDetector.pkl-style)
- Concatenated held-out test-split predictions for DeepBGC and ClusterFinder per fold
- cv_roc_auc.csv with per-fold and mean ROC AUC values for both detectors (expected: DeepBGC 0.984 vs. ClusterFinder 0.936)

## How to apply

Assemble a labeled training matrix from positive Pfam TSVs for the 1406 MIBiG BGCs and GeneSwap_Negatives.pfam.tsv (10128 negative clusters), manually adding an in_cluster column (1 for pfams inside a BGC, 0 outside) and a sequence_id that keeps each cluster's Pfam domains grouped. Load pfam2vec.csv embeddings and configure the Keras RNN detector from the JSON template: hidden_size=128, batch_size=16, timesteps=256, learning_rate=0.0001, num_epochs=1000, with early stopping on val_auc_roc (min_delta=0.0001, patience=20), shuffle=True, weighted=True. Run 10-fold cross-validation via deepbgc train, saving each fold's serialized detector, and train the ClusterFinder HMM on identical splits. Score each held-out test split with both detectors, concatenate predictions across folds, compute ROC AUC per model, and write per-fold and mean values to cv_roc_auc.csv. The rationale is that DeepBGC's BiLSTM over embedded Pfam domains captures long-range domain context that the ClusterFinder HMM misses.

## Related tools

- **DeepBGC** (BiLSTM RNN detector trained via deepbgc train on Pfam TSV examples with pfam2vec embeddings) — https://github.com/Merck/deepbgc
- **ClusterFinder** (Hidden Markov Model baseline detector trained on identical 10-fold splits for AUC comparison)
- **pfam2vec** (word2vec-like skip-gram embedding of Pfam domains used as detector input features (pfam2vec.csv)) — https://github.com/Merck/deepbgc
- **bgc-pipeline** (DVC-managed code and data for DeepBGC development and ClusterFinder/antiSMASH evaluation) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc train --detector myDetector. --train-pfam mibig_positives.pfam.tsv geneswap_negatives.pfam.tsv
```

## Evaluation signals

- cv_roc_auc.csv exists and contains an ROC AUC entry for both DeepBGC and ClusterFinder with values in [0,1]
- DeepBGC average ROC AUC exceeds the ClusterFinder average, approaching the reported 0.984 vs. 0.936
- Early stopping on val_auc_roc triggers (min_delta=0.0001, patience=20) so training does not exhaust 1000 epochs without improvement
- Every training row carries a binary in_cluster label and a sequence_id that keeps each cluster's Pfam domains grouped together
- Each fold produces a serialized detector that loads and scores the held-out test split without error

## Limitations

- Training data is heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting detection of novel BGCs in poorly characterized bacteria
- Artificial negatives (GeneSwap) differ from real non-BGC genomic context, so cross-validated AUC may overestimate real-genome discrimination performance
- The model may not extrapolate to BGC classes absent from the training set despite leave-class-out evidence of generalization

## Evidence
