---
name: multi-tool-prediction-overlap-analysis
description: Use when you need to compare BGC prediction counts and set overlap between
  DeepBGC, antiSMASH, and ClusterFinder on a common genome panel to expose differences
  in detection stringency.
when_to_use_negative:
- You only need BGC predictions from a single tool on one genome rather than a cross-tool
  benchmark on a common genome panel
- Your goal is BGC product-class or activity classification (use the DeepBGC random
  forest classifiers) rather than detection-count comparison
- Input genomes lack RefSeq annotation and the comparison requires standardized gene
  calls inconsistent with the paper's panel
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: DeepBGC
  role: BiLSTM RNN + pfam2vec BGC detector run via 'deepbgc pipeline' with strict
    2% FPR domain-score cutoff
  repo: https://github.com/Merck/deepbgc
- name: antiSMASH
  role: Rule-based BGC detector run with default settings for comparison baseline
- name: ClusterFinder
  role: HMM-based BGC detector run with default lenient settings as lenient baseline
- name: Prodigal
  role: Protein-coding gene prediction run automatically during the DeepBGC pipeline
    when genomes are not annotated
  repo: https://github.com/hyattpd/Prodigal/releases
- name: HMMER
  role: hmmscan used for Pfam domain detection during the DeepBGC pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:a2047b40318374ecc4f590ed96125b0283cfb9644ca38de4bab8c66d82eed4cd
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
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multi-tool-prediction-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that applies DeepBGC (strict 2% FPR cutoff), antiSMASH (default rule-based settings), and ClusterFinder (default lenient HMM settings) to the same set of 3376 RefSeq bacterial genomes and quantifies per-tool BGC counts plus pairwise prediction overlap, reproducing Figure 5A of the DeepBGC paper.

## When to use

Use when you need to compare BGC prediction counts and set overlap between DeepBGC, antiSMASH, and ClusterFinder on a common genome panel to expose differences in detection stringency.

## When NOT to use

- You only need BGC predictions from a single tool on one genome rather than a cross-tool benchmark on a common genome panel
- Your goal is BGC product-class or activity classification (use the DeepBGC random forest classifiers) rather than detection-count comparison
- Input genomes lack RefSeq annotation and the comparison requires standardized gene calls inconsistent with the paper's panel

## Inputs

- 3376 RefSeq bacterial genome sequence files (FASTA/GenBank)
- DeepBGC trained detector models and Pfam database (via deepbgc download)
- DeepBGC detector score cutoff calibrated to a 2% false positive rate
- antiSMASH and ClusterFinder default parameter configurations

## Outputs

- Per-genome BGC counts for DeepBGC, antiSMASH, and ClusterFinder
- Pairwise overlap fractions of predicted BGC sequence between tools
- figure5A_bgc_overlap.csv summary table

## How to apply

Download the 3376 RefSeq bacterial genome sequences and fetch the DeepBGC trained detector models and Pfam database with 'deepbgc download'. Run 'deepbgc pipeline' on each genome (Prodigal gene calling and HMMER hmmscan Pfam domain detection run automatically if not annotated) applying the strict detector domain-score cutoff corresponding to a 2% false positive rate. Run antiSMASH on the same genomes with default rule-based settings and ClusterFinder with its default lenient HMM settings. Aggregate per-genome BGC counts for each tool and compute the fraction of predicted BGC sequence shared between each pair of tools. Validate by checking counts against Figure 5A: ClusterFinder 62491 BGCs (>4.5x antiSMASH 13865, >5.5x DeepBGC 10926), ~90% of DeepBGC predictions overlapping ClusterFinder, ~39% overlapping antiSMASH (35% both), and ~5% (566) unique to DeepBGC.

## Related tools

- **DeepBGC** (BiLSTM RNN + pfam2vec BGC detector run via 'deepbgc pipeline' with strict 2% FPR domain-score cutoff) — https://github.com/Merck/deepbgc
- **antiSMASH** (Rule-based BGC detector run with default settings for comparison baseline)
- **ClusterFinder** (HMM-based BGC detector run with default lenient settings as lenient baseline)
- **Prodigal** (Protein-coding gene prediction run automatically during the DeepBGC pipeline when genomes are not annotated) — https://github.com/hyattpd/Prodigal/releases
- **HMMER** (hmmscan used for Pfam domain detection during the DeepBGC pipeline)

## Examples

```
deepbgc download && deepbgc pipeline genome.fa --detector deepbgc
```

## Evaluation signals

- DeepBGC predicts 10926 BGCs, antiSMASH 13865, and ClusterFinder 62491 across the 3376 RefSeq genomes
- ClusterFinder count is >4.5x antiSMASH and >5.5x DeepBGC
- ~90% of DeepBGC predictions overlap ClusterFinder; ~39% overlap antiSMASH (35% overlap both)
- 566 DeepBGC predictions (~5%) are unique to DeepBGC
- Reproduced per-tool counts and pairwise overlap fractions match Figure 5A of the publication

## Limitations

- ClusterFinder's default lenient settings inflate prediction counts relative to rule-based antiSMASH, so raw counts are not comparable measures of accuracy
- DeepBGC was trained on existing BGC databases heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces
- Some BGC classes were mis-identified in the S. coelicolor reference genome example, and recovery of some classes is threshold-dependent

## Evidence

- [readme] This repository provides data and examples that were used for development of DeepBGC and its evaluation with ClusterFinder and antiSMASH.: [evidence span withheld — anchor status: external_doc]
