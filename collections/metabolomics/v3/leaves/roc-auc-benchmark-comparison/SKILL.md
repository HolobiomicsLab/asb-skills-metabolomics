---
name: roc-auc-benchmark-comparison
description: Use when two or more BGC detectors must be quantitatively compared on
  labeled positives (e.g., 65 experimentally validated BGCs or MIBiG clusters) and
  negative regions, using per-domain or per-cluster prediction scores.
when_to_use_negative:
- When the goal is BGC discovery in a single genome rather than a quantitative detector
  comparison (use 'deepbgc pipeline' directly instead)
- When comparing rule-based tools like antiSMASH that output discrete region predictions
  without per-domain scores — ROC AUC requires continuous prediction scores
- When no labeled positive set (validated BGCs or MIBiG) is available to define the
  positive class
tools:
- name: DeepBGC
  role: Detector under evaluation; provides per-domain scores and is trained with
    'deepbgc train' under 10-fold cross-validation
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: Baseline HMM detector scored on identical domain sequences and identical cross-validation
    splits
- name: pfam2vec
  role: Word2vec-like Pfam domain embedding feeding the BiLSTM detector during training
    and scoring
- name: Prodigal
  role: Unsupervised protein-coding gene prediction supplying proteins for Pfam domain
    detection
  repo: https://github.com/hyattpd/Prodigal
- name: HMMER
  role: Detects Pfam domains that serve as the unit of analysis for ROC curves
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:be20cea70545b70187bab56c9a56e186cb84d15454b4ad5e493b576d9156563b
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
  - HMMER
  - Prodigal
  - TensorFlow
  - pfam2vec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# roc-auc-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for comparing BGC detection models (DeepBGC vs ClusterFinder variants) by computing domain-level and cluster-level ROC AUC scores on validated positive sets and artificial negatives, as done in the DeepBGC paper (Nucleic Acids Research, 2019).

## When to use

Use when two or more BGC detectors must be quantitatively compared on labeled positives (e.g., 65 experimentally validated BGCs or MIBiG clusters) and negative regions, using per-domain or per-cluster prediction scores.

## When NOT to use

- When the goal is BGC discovery in a single genome rather than a quantitative detector comparison (use 'deepbgc pipeline' directly instead)
- When comparing rule-based tools like antiSMASH that output discrete region predictions without per-domain scores — ROC AUC requires continuous prediction scores
- When no labeled positive set (validated BGCs or MIBiG) is available to define the positive class

## Inputs

- Genome FASTA files (six bacterial genomes containing the 65 experimentally validated BGCs)
- MIBiG positive Pfam TSVs (1406 BGCs) with in_cluster labels
- GeneSwap_Negatives.pfam.tsv (10128 artificial negative clusters)
- DeepBGC trained detector models and Pfam database (deepbgc download)
- ClusterFinder HMM detector files (original and retrained, as custom --detector input)
- pfam2vec.csv embedding table

## Outputs

- Domain-level ROC curves and AUC ROC values per detector (target: DeepBGC 0.923, ClusterFinder 0.847)
- cv_roc_auc.csv with per-fold and mean cross-validation ROC AUC for DeepBGC and ClusterFinder (target: 0.984 vs 0.936)
- Per-fold trained detector files (serialized model per fold)

## How to apply

Fetch trained detector models and the Pfam database with 'deepbgc download' and verify with 'deepbgc info'. Run 'deepbgc pipeline' on the target genome FASTA files so Prodigal calls proteins and HMMER detects Pfam domains automatically; score the same domains with the original ClusterFinder HMM and any retrained detector via a custom --detector file. For positional-accuracy evaluation, label each Pfam domain positive if it lies within one of the 65 experimentally validated BGCs from six bacterial genomes and negative otherwise, then compute domain-level ROC AUC from per-domain scores (target: DeepBGC 0.923 vs ClusterFinder 0.847). For discrimination evaluation, assemble a training matrix of Pfam TSVs (1406 MIBiG positives plus GeneSwap_Negatives.pfam.tsv with 10128 negative clusters) with a binary in_cluster column, run 10-fold cross-validation with 'deepbgc train' (KerasRNN: hidden_size=128, batch_size=16, timesteps=256, learning_rate=0.0001, early stopping on val_auc_roc with patience=20), score each held-out fold with both detectors, concatenate test predictions, and compute ROC AUC per fold and mean (target: DeepBGC 0.984 vs ClusterFinder 0.936), writing results to cv_roc_auc.csv.

## Related tools

- **DeepBGC** (Detector under evaluation; provides per-domain scores and is trained with 'deepbgc train' under 10-fold cross-validation) — https://github.com/Merck/deepbgc
- **ClusterFinder** (Baseline HMM detector scored on identical domain sequences and identical cross-validation splits)
- **pfam2vec** (Word2vec-like Pfam domain embedding feeding the BiLSTM detector during training and scoring)
- **Prodigal** (Unsupervised protein-coding gene prediction supplying proteins for Pfam domain detection) — https://github.com/hyattpd/Prodigal
- **HMMER** (Detects Pfam domains that serve as the unit of analysis for ROC curves)

## Examples

```
deepbgc pipeline mySequence.fa
```

## Evaluation signals

- DeepBGC domain-level AUC ROC on the 65 validated BGC set reproduces 0.923 and ClusterFinder reproduces 0.847
- cv_roc_auc.csv exists and contains an ROC AUC entry for both DeepBGC and ClusterFinder, each within [0,1]
- DeepBGC average cross-validation ROC AUC (0.984) exceeds ClusterFinder (0.936) on MIBiG positives vs GeneSwap negatives
- Every Pfam domain carries a binary in_cluster label (1 inside a validated BGC, 0 outside) and a sequence_id keeping each cluster's domains grouped
- Test-split predictions are concatenated across all 10 folds before computing ROC AUC, with per-fold values reported alongside the mean

## Limitations

- Training data bias: existing BGC databases are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting evaluation of novel BGC detection in poorly characterized bacteria
- Artificial GeneSwap negatives do not represent real non-BGC sequence, so cross-validation AUC may overstate discrimination of genuine backgrounds
- Threshold-dependence: some well-studied BGC classes are recovered regardless of threshold while others are more threshold-dependent, so AUC alone does not capture class-level recovery

## Evidence
