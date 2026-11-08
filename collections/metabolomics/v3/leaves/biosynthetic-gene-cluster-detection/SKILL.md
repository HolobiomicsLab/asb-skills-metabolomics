---
name: biosynthetic-gene-cluster-detection
description: Use when mining assembled bacterial (or fungal) genome FASTA files for
  BGCs and needing reduced false positive rates or detection of BGC classes unseen
  during training, especially when ClusterFinder or antiSMASH yields too many loose
  predictions.
when_to_use_negative:
- The input is already a Pfam feature table or curated BGC training matrix — use 'deepbgc
  prepare'/'deepbgc train' rather than the genome-mining pipeline
- You need rule-based annotation of known biosynthetic product types rather than ML-based
  detection — antiSMASH default settings are the appropriate comparison method
- 'You require confident product-class assignment: in this study the majority of novel
  BGC classes detected could not be confidently assigned'
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0797
tools:
- name: DeepBGC
  role: Primary BiLSTM + pfam2vec detector and random-forest classifier invoked through
    the deepbgc pipeline command
  repo: https://github.com/Merck/deepbgc
- name: Prodigal
  role: Unsupervised protein-coding gene prediction supplying proteins for Pfam domain
    annotation inside the DeepBGC pipeline
  repo: https://github.com/hyattpd/Prodigal
- name: HMMER
  role: hmmscan-based Pfam domain detection on predicted proteins feeding the detector's
    Pfam TSV input
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM-based BGC detection baseline (original and retrained variants) used for
    comparison of precision and AUC
  repo: https://github.com/Merck/bgc-pipeline
- name: antiSMASH
  role: Rule-based BGC detection baseline run with default settings for counts/overlap
    comparison on the 3376 RefSeq genomes
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_008
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:e9bf96e27828bdf2c04f462e06de60e015b8c38ff04dc32c08a90e8dad7acd57
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
  - Keras
  - Prodigal
  - TensorFlow
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# biosynthetic-gene-cluster-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect and classify biosynthetic gene clusters (BGCs) in bacterial genomes using DeepBGC's BiLSTM recurrent neural network over pfam2vec-embedded Pfam domain sequences, a deep-learning alternative to ClusterFinder's HMM and antiSMASH's rule-based detection.

## When to use

Use when mining assembled bacterial (or fungal) genome FASTA files for BGCs and needing reduced false positive rates or detection of BGC classes unseen during training, especially when ClusterFinder or antiSMASH yields too many loose predictions.

## When NOT to use

- The input is already a Pfam feature table or curated BGC training matrix — use 'deepbgc prepare'/'deepbgc train' rather than the genome-mining pipeline
- You need rule-based annotation of known biosynthetic product types rather than ML-based detection — antiSMASH default settings are the appropriate comparison method
- You require confident product-class assignment: in this study the majority of novel BGC classes detected could not be confidently assigned

## Inputs

- Bacterial or fungal genome FASTA file(s) (assembled contigs, e.g. RefSeq genomes)
- Optionally pre-annotated Pfam TSV domain tables (with in_cluster and sequence_id columns)
- DeepBGC trained detector models and Pfam database fetched via 'deepbgc download'
- pfam2vec.csv embedding vectors (from the DeepBGC v0.1.0 release) for custom training

## Outputs

- Per-genome BGC region predictions with domain-level scores (DeepBGC pipeline output folder)
- BGC product class and activity classifications from the random forest classifier
- antismash. for visualization of DeepBGC regions alongside antiSMASH on the antiSMASH site
- Summary tables such as figure5A_bgc_overlap.csv (per-tool BGC counts and pairwise overlap fractions) or bgc_precision_comparison.csv

## How to apply

Install DeepBGC (bioconda/pip), run 'deepbgc download' to fetch the trained detector models and Pfam database, and confirm dependencies with 'deepbgc info'. Run 'deepbgc pipeline' on each genome FASTA; Prodigal calls protein-coding genes and HMMER hmmscan annotates Pfam domains automatically if the input is unannotated. Detection uses a BiLSTM RNN over Pfam domains encoded as pfam2vec vectors (hidden_size 128, timesteps 256, adam optimizer, learning rate 0.0001), with domain-level scores thresholdable at calibrated operating points such as a strict 2% false-positive-rate cutoff or a lenient 80% true-positive-rate cutoff. Post-processing merges thresholded domain calls into discrete BGC regions, which substantially raises BGC-level precision at lenient thresholds. Detected BGCs are then classified by product class using an accompanying random forest classifier on Pfam domain composition. For detection across many genomes, apply the same pipeline to each genome and aggregate per-genome BGC counts and pairwise overlap between tools (e.g. DeepBGC vs antiSMASH vs ClusterFinder).

## Related tools

- **DeepBGC** (Primary BiLSTM + pfam2vec detector and random-forest classifier invoked through the deepbgc pipeline command) — https://github.com/Merck/deepbgc
- **Prodigal** (Unsupervised protein-coding gene prediction supplying proteins for Pfam domain annotation inside the DeepBGC pipeline) — https://github.com/hyattpd/Prodigal
- **HMMER** (hmmscan-based Pfam domain detection on predicted proteins feeding the detector's Pfam TSV input) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM-based BGC detection baseline (original and retrained variants) used for comparison of precision and AUC) — https://github.com/Merck/bgc-pipeline
- **antiSMASH** (Rule-based BGC detection baseline run with default settings for counts/overlap comparison on the 3376 RefSeq genomes)

## Examples

```
deepbgc download && deepbgc pipeline mySequence.fa
```

## Evaluation signals

- Domain-level ROC AUC on the 65 experimentally validated BGCs across six genomes should reach ~0.923 for DeepBGC vs ~0.847 for ClusterFinder
- Bootstrap evaluation over the nine manually annotated genomes should reproduce DeepBGC AUC 0.946 / average precision 0.75, vs original ClusterFinder 0.837 / 0.28 and retrained ClusterFinder 0.912 / 0.63
- BGC-level precision on the nine annotated genomes at the 10% FPR cutoff should be ~34% (DeepBGC) vs ~26% (ClusterFinder), and at the 80% TPR cutoff ~44% vs ~9%
- Across 3376 RefSeq genomes, DeepBGC (strict 2% FPR cutoff) should predict ~10926 BGCs vs antiSMASH 13865 and ClusterFinder 62491, with ~90% of DeepBGC predictions overlapping ClusterFinder and ~39% overlapping antiSMASH
- Verify 'deepbgc info' lists all downloaded dependencies and models before running the pipeline, and that BGC count outputs are non-negative integers with per-class AUC values in [0,1]

## Limitations

- Training data bias: the model was trained on BGC databases heavily biased towards natural-product 'workhorses' such as Streptomyces, limiting novel BGC detection in poorly characterized bacteria
- Some BGC classes were mis-identified in the S. coelicolor reference genome example, and class recovery is threshold-dependent for some well-studied classes
- The majority of novel BGC classes identified could not be confidently assigned; more diverse BGC databases are needed for improved training and validation

## Evidence
