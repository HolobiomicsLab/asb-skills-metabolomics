---
name: bgc-region-post-processing
description: Use when per-domain BGC detector scores (DeepBGC BiLSTM or ClusterFinder
  HMM) must be thresholded at a calibrated operating point (10% FPR or 80% TPR) and
  collapsed into discrete BGC regions for genome-scale precision evaluation.
when_to_use_negative:
- You only need raw per-domain scores for training or feature extraction — region
  post-processing discards per-domain score detail
- You are comparing against antiSMASH's rule-based BGC boundaries, whose region definition
  is not produced by threshold-and-merge post-processing of domain scores
- The detector has already emitted final BGC regions (e.g. deepbgc pipeline's own
  region output) and you are not evaluating the merging step itself
tools:
- name: DeepBGC
  role: Produces the per-domain BiLSTM detection scores that are thresholded and merged
    into BGC regions; also provides the pipeline that performs post-processing
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM baseline whose thresholded domain calls are post-processed identically
    for the 4-condition precision comparison
- name: HMMER
  role: Runs hmmscan to annotate input genomes with Pfam domains, the units being
    thresholded and merged
- name: Prodigal
  role: Predicts protein-coding genes so per-gene Pfam domain compositions can be
    scored by the detector
  repo: https://github.com/hyattpd/Prodigal/releases
- name: pfam2vec
  role: Provides the word2vec-like Pfam embedding consumed by the DeepBGC detector
    whose output is post-processed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:c3507101d7b6d1347c3b63a19c09c35cb57e4c7913c5f6c036657259a9ae7322
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# bgc-region-post-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-processing skill from the DeepBGC paper: thresholded per-domain Pfam scores are converted into discrete BGC regions, with an optional region-merging step whose effect on BGC-level precision is explicitly benchmarked against ClusterFinder on nine manually annotated bacterial genomes.

## When to use

Use when per-domain BGC detector scores (DeepBGC BiLSTM or ClusterFinder HMM) must be thresholded at a calibrated operating point (10% FPR or 80% TPR) and collapsed into discrete BGC regions for genome-scale precision evaluation.

## When NOT to use

- You only need raw per-domain scores for training or feature extraction — region post-processing discards per-domain score detail
- You are comparing against antiSMASH's rule-based BGC boundaries, whose region definition is not produced by threshold-and-merge post-processing of domain scores
- The detector has already emitted final BGC regions (e.g. deepbgc pipeline's own region output) and you are not evaluating the merging step itself

## Inputs

- Per-domain Pfam detection scores from the DeepBGC BiLSTM detector (deepbgc pipeline output on FASTA genomes)
- Per-domain scores from the ClusterFinder HMM baseline
- Domain-level score thresholds calibrated at the 10% FPR and 80% TPR operating points
- Manual BGC annotations for the nine manually annotated bacterial genomes
- Pfam domain annotations (HMMER hmmscan) and Prodigal gene predictions

## Outputs

- Discrete predicted BGC regions, generated both with and without the region-merging post-processing step
- Matched prediction-vs-annotation region comparisons
- bgc_precision_comparison.csv containing BGC-level precision across the 4 conditions (DeepBGC vs ClusterFinder × 10% FPR vs 80% TPR)

## How to apply

Run the DeepBGC pipeline (`deepbgc download`, then `deepbgc pipeline`) on each genome to obtain per-domain detection scores, using HMMER hmmscan for Pfam annotation and Prodigal for gene prediction. Apply domain-level score thresholds calibrated at the stringent 10% false positive rate and lenient 80% true positive rate operating points to the per-domain predictions. Convert thresholded domain calls into discrete BGC regions twice — once with the post-processing (region merging) step and once without — to isolate its effect on BGC-level precision. Match predicted BGC regions against manual annotations and compute BGC-level precision for both DeepBGC and the ClusterFinder HMM baseline under each threshold and post-processing condition. The key benchmark: at 10% FPR, precision is comparable (DeepBGC 34% vs ClusterFinder 26%), but at the lenient 80% TPR threshold, post-processed DeepBGC regions retain 44% precision versus 9% for ClusterFinder (>4-fold higher), demonstrating that lenient thresholds need the merging step to avoid fragmenting or inflating false regions.

## Related tools

- **DeepBGC** (Produces the per-domain BiLSTM detection scores that are thresholded and merged into BGC regions; also provides the pipeline that performs post-processing) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM baseline whose thresholded domain calls are post-processed identically for the 4-condition precision comparison)
- **HMMER** (Runs hmmscan to annotate input genomes with Pfam domains, the units being thresholded and merged)
- **Prodigal** (Predicts protein-coding genes so per-gene Pfam domain compositions can be scored by the detector) — https://github.com/hyattpd/Prodigal/releases
- **pfam2vec** (Provides the word2vec-like Pfam embedding consumed by the DeepBGC detector whose output is post-processed)

## Examples

```
deepbgc download && deepbgc info && deepbgc pipeline mySequence.fa
```

## Evaluation signals

- Reproduced BGC-level precision at the stringent 10% FPR cutoff matches the reported 34% (DeepBGC) vs 26% (ClusterFinder)
- Reproduced BGC-level precision at the lenient 80% TPR cutoff matches the reported 44% (DeepBGC) vs 9% (ClusterFinder), i.e. >4-fold improvement
- The with-merging vs without-merging variants both run on identical thresholded domain calls, so region differences are attributable solely to the merging step
- bgc_precision_comparison.csv contains the full 2×2 grid (tool × threshold) with no missing cells
- `deepbgc info` confirms trained detector models and the Pfam database were downloaded before running the pipeline

## Limitations

- Training data were heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting precision on poorly characterized bacteria
- Some well-studied BGC classes were recovered regardless of threshold while others were more threshold-dependent, so region recovery is not uniform across classes
- Some BGC classes were mis-identified in the S. coelicolor reference genome example, so region matching to manual annotations can disagree at class level even when region detection succeeds

## Evidence
