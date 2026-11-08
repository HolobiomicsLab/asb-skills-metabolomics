---
name: precision-benchmarking-against-manual-annotations
description: Use when comparing BGC detection tools on genomes that have manual BGC
  annotations, and you need BGC-level precision at calibrated domain-score thresholds
  (e.g. 10% FPR vs 80% TPR) rather than only domain-level ROC metrics.
when_to_use_negative:
- When no manually annotated ground-truth BGCs exist for the test genomes — precision
  cannot be computed without manual annotations.
- When the goal is product-class or activity classification accuracy (use random forest
  classifier evaluation instead of BGC-region precision).
- When only domain-level (per-Pfam) performance is needed; this skill targets BGC-region-level
  precision after thresholding and region merging.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: DeepBGC
  role: Candidate BiLSTM detector whose pipeline produces per-domain BGC scores that
    are thresholded and converted to BGC regions for precision benchmarking
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM-based baseline predictor whose BGC-level precision is compared against
    DeepBGC at both operating points
- name: HMMER
  role: hmmscan annotates predicted genes with Pfam domains used as input features
    for scoring
- name: Prodigal
  role: Performs gene prediction on each genome prior to Pfam domain annotation
  repo: https://github.com/hyattpd/Prodigal/releases
- name: bgc-pipeline
  role: Provides data and evaluation code (DVC-managed) used to develop and evaluate
    DeepBGC against ClusterFinder
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:55077ed9188313d87474cee3a57c828bf0162ad5438d11aee1984ead14d5686d
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
  - bgc-pipeline
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# precision-benchmarking-against-manual-annotations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark BGC-level precision of a predictor (e.g. DeepBGC) against a baseline (e.g. ClusterFinder) by matching thresholded predicted regions to manually annotated BGCs on nine curated bacterial genomes, at both a stringent 10% FPR and a lenient 80% TPR domain-score operating point, with and without region post-processing.

## When to use

Use when comparing BGC detection tools on genomes that have manual BGC annotations, and you need BGC-level precision at calibrated domain-score thresholds (e.g. 10% FPR vs 80% TPR) rather than only domain-level ROC metrics.

## When NOT to use

- When no manually annotated ground-truth BGCs exist for the test genomes — precision cannot be computed without manual annotations.
- When the goal is product-class or activity classification accuracy (use random forest classifier evaluation instead of BGC-region precision).
- When only domain-level (per-Pfam) performance is needed; this skill targets BGC-region-level precision after thresholding and region merging.

## Inputs

- Nine manually annotated bacterial genomes (FASTA)
- Per-domain prediction scores from DeepBGC pipeline runs (Pfam TSV)
- ClusterFinder HMM baseline predictions
- Domain-level score thresholds calibrated at 10% FPR and 80% TPR
- Manual BGC annotations (ground-truth cluster coordinates)

## Outputs

- bgc_precision_comparison.csv: 4-condition BGC-level precision table (DeepBGC vs ClusterFinder x 10% FPR vs 80% TPR)
- Matched predicted-vs-manual BGC region sets with and without post-processing

## How to apply

Run the candidate predictor's genome pipeline (e.g. 'deepbgc pipeline' using HMMER hmmscan for Pfam annotation and Prodigal for gene prediction) on each of the nine manually annotated genomes to obtain per-domain prediction scores. Apply two calibrated domain-level thresholds: the stringent 10% false-positive-rate operating point and the lenient 80% true-positive-rate operating point. Convert thresholded domain calls into discrete BGC regions twice — once with post-processing (region merging) and once without — to isolate its effect on precision. Match predicted BGC regions against the manual annotations and compute BGC-level precision for the tool and for the ClusterFinder HMM baseline under each threshold/post-processing combination, then write the 4-condition comparison (tool vs baseline x 10% FPR vs 80% TPR) to a CSV. Validation requires reproducing the reported precisions: 34% vs 26% at 10% FPR and 44% vs 9% at 80% TPR, where DeepBGC exceeds 4-fold precision over ClusterFinder under the lenient threshold.

## Related tools

- **DeepBGC** (Candidate BiLSTM detector whose pipeline produces per-domain BGC scores that are thresholded and converted to BGC regions for precision benchmarking) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM-based baseline predictor whose BGC-level precision is compared against DeepBGC at both operating points)
- **HMMER** (hmmscan annotates predicted genes with Pfam domains used as input features for scoring)
- **Prodigal** (Performs gene prediction on each genome prior to Pfam domain annotation) — https://github.com/hyattpd/Prodigal/releases
- **bgc-pipeline** (Provides data and evaluation code (DVC-managed) used to develop and evaluate DeepBGC against ClusterFinder) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc download; deepbgc pipeline genome1.fa
```

## Evaluation signals

- Reproduced BGC-level precisions match the reported values: 34% (DeepBGC) vs 26% (ClusterFinder) at the 10% FPR threshold.
- At the 80% TPR threshold, DeepBGC precision reaches 44% while ClusterFinder falls to 9% (>4-fold difference).
- Precision is computed for both post-processing conditions (region merging on and off) to quantify its effect.
- The 4-condition comparison table is complete (tool vs baseline x 10% FPR vs 80% TPR) and written to bgc_precision_comparison.csv.
- Matched regions are evaluated against the same nine manually annotated genomes used in the original study.

## Limitations

- Precision under the lenient 80% TPR threshold is highly sensitive to the post-processing (region merging) step, so results must be reported with and without it.
- Domain-score thresholds must be recalibrated (10% FPR and 80% TPR operating points) for any changed model or corpus; stale thresholds invalidate the comparison.
- Training data bias towards Streptomyces 'workhorse' BGCs limits generalization of benchmark results to poorly characterized bacteria and novel BGC classes.

## Evidence

- [results] Under the stringent 10% FPR cutoff on the manually annotated genomes, BGC-level precision was comparable between DeepBGC and ClusterFinder (34% vs 26%): [evidence span withheld — anchor status: not_found]
- [results] ClusterFinder's lenient-threshold predictions fell to 9% precision while DeepBGC reached 44% (>4-fold higher): [evidence span withheld — anchor status: not_found]
- [readme] This repository provides data and examples that were used for development of DeepBGC and its evaluation with ClusterFinder and antiSMASH.: [evidence span withheld — anchor status: external_doc]
