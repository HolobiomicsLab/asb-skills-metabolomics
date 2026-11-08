---
name: pfam-domain-level-prediction-scoring
description: Use when benchmarking BGC prediction methods at per-Pfam-domain resolution,
  e.g. comparing DeepBGC's deep-learning detector against ClusterFinder HMMs using
  AUC ROC on validated BGC labels.
when_to_use_negative:
- Input is already a BGC-level (cluster-level) prediction table rather than per-Pfam-domain
  scores — the skill requires domain-level granularity
- The genome set lacks experimentally validated BGC annotations, so positive/negative
  domain labels cannot be assigned
- The goal is rule-based BGC detection comparison (e.g. antiSMASH), which does not
  produce per-domain probability scores for ROC analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_0157
tools:
- name: DeepBGC
  role: Deep-learning (BiLSTM RNN with pfam2vec embeddings) detector producing per-domain
    BGC prediction scores
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: Original HMM-based detector whose per-domain scores are the baseline for AUC
    comparison
- name: HMMER
  role: Detects Pfam protein domains in Prodigal-predicted proteins during the deepbgc
    pipeline
- name: Prodigal
  role: Predicts protein-coding genes from the genome FASTA before Pfam domain annotation
  repo: https://github.com/hyattpd/Prodigal/releases
- name: pfam2vec
  role: word2vec-like embedding of Pfam domains consumed by the DeepBGC scoring model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:0f077ae1b97667081a08f921e0975b2f26e86eee9a2e696104d460e18a55bb88
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
  - pfam2vec
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pfam-domain-level-prediction-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score each Pfam protein domain in a genome with BGC-detection models (DeepBGC BiLSTM, ClusterFinder HMM, retrained ClusterFinder) and evaluate performance with domain-level ROC curves and AUC against the 65 experimentally validated BGCs in six bacterial genomes.

## When to use

Use when benchmarking BGC prediction methods at per-Pfam-domain resolution, e.g. comparing DeepBGC's deep-learning detector against ClusterFinder HMMs using AUC ROC on validated BGC labels.

## When NOT to use

- Input is already a BGC-level (cluster-level) prediction table rather than per-Pfam-domain scores — the skill requires domain-level granularity
- The genome set lacks experimentally validated BGC annotations, so positive/negative domain labels cannot be assigned
- The goal is rule-based BGC detection comparison (e.g. antiSMASH), which does not produce per-domain probability scores for ROC analysis

## Inputs

- Six bacterial genome FASTA files containing the 65 experimentally validated BGCs
- DeepBGC trained detector models and Pfam database (fetched with deepbgc download)
- ClusterFinder HMM and retrained ClusterFinder detector files (passed via --detector)
- Per-domain label assignments (in_cluster: 1 for Pfam domains inside a validated BGC, 0 otherwise)

## Outputs

- Per-Pfam-domain prediction scores for each detector (DeepBGC, ClusterFinder, retrained ClusterFinder)
- Domain-level ROC curves per tool
- AUC ROC values per tool (expected: DeepBGC 0.923, ClusterFinder 0.847)

## How to apply

Run the deepbgc pipeline on each of the six genome FASTA files so Prodigal calls proteins and HMMER detects Pfam domains, then score the same per-domain Pfam sequences with the original ClusterFinder HMM and the retrained ClusterFinder model loaded as custom detector files via --detector. Label each Pfam domain as positive if it lies within one of the 65 experimentally validated BGCs and negative if it falls in an unannotated region. Compute domain-level ROC curves and AUC ROC for each tool from their per-domain prediction scores. The rationale is that domain-level scoring directly measures each detector's ability to separate BGC-resident Pfam domains from background domains, enabling head-to-head comparison of heterogeneous detectors on identical inputs. Validate by comparing computed AUC values to published figures (DeepBGC 0.923 vs ClusterFinder 0.847).

## Related tools

- **DeepBGC** (Deep-learning (BiLSTM RNN with pfam2vec embeddings) detector producing per-domain BGC prediction scores) — https://github.com/Merck/deepbgc
- **ClusterFinder** (Original HMM-based detector whose per-domain scores are the baseline for AUC comparison)
- **HMMER** (Detects Pfam protein domains in Prodigal-predicted proteins during the deepbgc pipeline)
- **Prodigal** (Predicts protein-coding genes from the genome FASTA before Pfam domain annotation) — https://github.com/hyattpd/Prodigal/releases
- **pfam2vec** (word2vec-like embedding of Pfam domains consumed by the DeepBGC scoring model)

## Examples

```
deepbgc pipeline mySequence.fa
```

## Evaluation signals

- Computed AUC ROC for DeepBGC reproduces the published 0.923 on the 65 validated BGC set
- Computed AUC ROC for the original ClusterFinder HMM reproduces the published 0.847
- DeepBGC's domain-level ROC curve dominates (outperforms) both ClusterFinder variants across thresholds
- Every Pfam domain in the six genomes carries a valid in_cluster label (0 or 1) before ROC computation
- deepbgc info confirms trained detector models and Pfam database were downloaded before scoring

## Limitations

- The validated-BGC benchmark comprises only 65 BGCs across six bacterial genomes, limiting generalizability
- The DeepBGC model was trained on existing BGC databases heavily biased towards Streptomyces 'workhorse' BGCs
- Detection performance varies by threshold: some well-studied classes were recovered regardless of threshold while others were more threshold-dependent

## Evidence

- [readme] Evaluation repo purpose: [evidence span withheld — anchor status: external_doc]
