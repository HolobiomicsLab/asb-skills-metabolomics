---
name: pfam2vec-embedding-vectorization
description: 'Use when preparing Pfam-domain training or scoring inputs for the DeepBGC
  KerasRNN detector: rows of a Pfam TSV with in_cluster/sequence_id labels must be
  converted to pfam2vec.csv vectors before feeding the BiLSTM.'
when_to_use_negative:
- Input is raw nucleotide sequence rather than Pfam-annotated domain TSVs — run gene
  prediction (Prodigal) and domain annotation and `deepbgc prepare` first
- A rule-based BGC annotation comparison (e.g. antiSMASH-style) is the goal; pfam2vec
  serves DeepBGC's deep learning detector, not rule-based cluster delineation
- You only need ClusterFinder HMM scores, which do not consume pfam2vec embeddings
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0797
tools:
- name: DeepBGC
  role: Pipeline whose KerasRNN BiLSTM detector consumes pfam2vec vectors; `deepbgc
    prepare` builds the Pfam TSVs and `deepbgc train --config` accepts the pfam2vec.csv
    embeddings
  repo: https://github.com/Merck/deepbgc
- name: pfam2vec
  role: word2vec-like skip-gram neural network that produces the domain embedding
    vectors from an unlabeled genomic corpus
  repo: https://github.com/Merck/bgc-pipeline
- name: Prodigal
  role: Upstream protein-coding gene prediction whose proteins are domain-annotated
    to produce the Pfam domain sequences that get embedded
  repo: https://github.com/hyattpd/Prodigal
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:b9e0f1baf1fce9e2fb8d7f011b8e9f2ea61ebc4cecacca22f8c276fd08a4c076
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
  - Prodigal
  - TensorFlow
  - pfam2vec
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pfam2vec-embedding-vectorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Vectorize Pfam protein domain compositions of gene clusters using pfam2vec, a word2vec-like skip-gram embedding trained on a large unlabeled corpus of genomic sequences, so that a BiLSTM RNN detector (DeepBGC) can consume fixed-length numeric representations of BGC and non-BGC sequences.

## When to use

Use when preparing Pfam-domain training or scoring inputs for the DeepBGC KerasRNN detector: rows of a Pfam TSV with in_cluster/sequence_id labels must be converted to pfam2vec.csv vectors before feeding the BiLSTM.

## When NOT to use

- Input is raw nucleotide sequence rather than Pfam-annotated domain TSVs — run gene prediction (Prodigal) and domain annotation and `deepbgc prepare` first
- A rule-based BGC annotation comparison (e.g. antiSMASH-style) is the goal; pfam2vec serves DeepBGC's deep learning detector, not rule-based cluster delineation
- You only need ClusterFinder HMM scores, which do not consume pfam2vec embeddings

## Inputs

- Pfam TSV training examples with in_cluster (0/1) and sequence_id columns (e.g. MIBiG positives, GeneSwap_Negatives.pfam.tsv with 10128 negative clusters)
- pfam2vec.csv pre-trained embedding table (DeepBGC release v0.1.0)
- Unlabeled corpus of genomic Pfam domain sequences (for training new embeddings)
- KerasRNN detector JSON template with Pfam2VecTransformer + ProteinBorderTransformer feature configuration

## Outputs

- pfam2vec.csv vector embedding table mapping Pfam domains to dense vectors
- pfam2vec-encoded feature matrices consumed by the BiLSTM KerasRNN detector (myDetector.pkl serialized models)
- Trained DeepBGC detector whose predictions support cv_roc_auc.csv, leave_class_out_auc.csv, and detection_benchmark_metrics.csv

## How to apply

Generate or reuse pfam2vec embedding vectors (pfam2vec.csv) trained with a word2vec-like skip-gram neural network over a large unlabeled corpus of genomic Pfam-domain sequences; the pre-trained table can be downloaded from the DeepBGC v0.1.0 release. Prepare training examples in Pfam TSV format via `deepbgc prepare`, adding the binary in_cluster column (1 for Pfams inside a BGC, 0 outside) and a sequence_id column that keeps each cluster's Pfam domains grouped during shuffling. Supply the pfam2vec.csv vectors to `deepbgc train` via --config when training the KerasRNN detector (hidden_size 128, timesteps 256, learning_rate 0.0001, early stopping on val_auc_roc with patience 20 and min_delta 0.0001), together with Pfam2VecTransformer plus ProteinBorderTransformer input features. The rationale is that embedding domain co-occurrence semantics lets the recurrent model generalize to BGCs of known and unseen classes, as validated by 10-fold cross-validation (AUC 0.984 vs ClusterFinder 0.936) and leave-class-out validation (AUC 0.946 vs 0.865).

## Related tools

- **DeepBGC** (Pipeline whose KerasRNN BiLSTM detector consumes pfam2vec vectors; `deepbgc prepare` builds the Pfam TSVs and `deepbgc train --config` accepts the pfam2vec.csv embeddings) — https://github.com/Merck/deepbgc
- **pfam2vec** (word2vec-like skip-gram neural network that produces the domain embedding vectors from an unlabeled genomic corpus) — https://github.com/Merck/bgc-pipeline
- **Prodigal** (Upstream protein-coding gene prediction whose proteins are domain-annotated to produce the Pfam domain sequences that get embedded) — https://github.com/hyattpd/Prodigal

## Examples

```
deepbgc train --config myDetector. --matrix training.pfam.tsv --pfam2vec pfam2vec.csv --outdir detector_out
```

## Evaluation signals

- pfam2vec.csv loads as a dense vector table for every Pfam domain used in the training TSVs (no missing-domain errors during detector training)
- 10-fold cross-validation ROC AUC of the pfam2vec-fed DeepBGC detector exceeds ClusterFinder's, reproducing the reported 0.984 vs 0.936 on MIBiG positives vs GeneSwap negatives
- Leave-class-out AUC values per held-out MIBiG class lie in [0,1] and average 0.946 vs ClusterFinder's 0.865 across the six non-hybrid classes
- Bootstrap nine-genome benchmark reproduces DeepBGC AUC 0.946 / average precision 0.75 (vs ClusterFinder 0.837/0.28 and retrained ClusterFinder 0.912/0.63)
- Training log shows val_auc_roc improving monotonically until early stopping (patience 20, min_delta 0.0001) triggers before 1000 epochs

## Limitations

- Downstream model inherits training-data bias: existing BGC databases are heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting novel BGC detection in poorly characterized bacteria
- Using pfam2vec to annotate domains of unknown function was left incomplete and explicitly beyond the study scope
- The majority of novel BGC classes identified downstream could not be confidently assigned, despite improved detection

## Evidence
