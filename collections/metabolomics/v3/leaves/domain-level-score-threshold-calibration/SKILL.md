---
name: domain-level-score-threshold-calibration
description: 'Use when comparing BGC detection tools at matched domain-level stringency:
  you have per-domain prediction scores and labeled calibration data, and need cutoffs
  at defined FPR/TPR operating points before merging domains into clusters.'
when_to_use_negative:
- You only need product-class or activity classification of already-detected BGCs
  (use the random forest classifiers, not threshold calibration)
- Input is rule-based detector output such as antiSMASH cluster predictions that lack
  continuous per-domain scores to threshold
- No labeled genome annotations are available to estimate FPR/TPR operating points
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0154
tools:
- name: DeepBGC
  role: Produces per-Pfam-domain BiLSTM detection scores that are thresholded at the
    calibrated operating points
  repo: https://github.com/Merck/deepbgc
- name: ClusterFinder
  role: HMM baseline whose domain-level predictions are thresholded at the same operating
    points for the precision comparison
- name: HMMER
  role: Runs hmmscan to annotate Pfam domains that feed the domain-level scores
- name: Prodigal
  role: Predicts protein-coding genes upstream of Pfam annotation and domain scoring
  repo: https://github.com/hyattpd/Prodigal/releases
- name: bgc-pipeline
  role: Development and evaluation code used to reproduce the paper's calibration
    and precision-comparison analyses
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:2575fd80aac6d48a52dbd5f1b58c76239f028b6f91d1ebf7aa3cdf56fcb1ad27
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# domain-level-score-threshold-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibration of per-Pfam-domain detection score thresholds to fixed operating points — a stringent 10% false positive rate and a lenient 80% true positive rate — used in the DeepBGC paper to compare BGC-level precision against ClusterFinder on nine manually annotated bacterial genomes.

## When to use

Use when comparing BGC detection tools at matched domain-level stringency: you have per-domain prediction scores and labeled calibration data, and need cutoffs at defined FPR/TPR operating points before merging domains into clusters.

## When NOT to use

- You only need product-class or activity classification of already-detected BGCs (use the random forest classifiers, not threshold calibration)
- Input is rule-based detector output such as antiSMASH cluster predictions that lack continuous per-domain scores to threshold
- No labeled genome annotations are available to estimate FPR/TPR operating points

## Inputs

- Per-Pfam-domain detection scores for each of the nine manually annotated bacterial genomes (from DeepBGC BiLSTM detector or ClusterFinder HMM)
- Manually annotated BGC genomes with labeled in-cluster vs out-of-cluster domains for FPR/TPR calibration
- Detector dependencies: Pfam database and trained DeepBGC models (deepbgc download), HMMER hmmscan, Prodigal gene predictions

## Outputs

- Two calibrated domain-level score thresholds (10% FPR stringent, 80% TPR lenient)
- Thresholded binary domain calls per genome, converted to BGC regions with and without post-processing/merging
- bgc_precision_comparison.csv with BGC-level precision across the 4 conditions (tool x threshold)

## How to apply

Run each detector (DeepBGC pipeline, or the ClusterFinder HMM baseline) over the nine manually annotated genomes to obtain per-Pfam-domain scores; DeepBGC scores come from the BiLSTM detector after HMMER hmmscan Pfam annotation and Prodigal gene prediction. Using labeled domains from the manual annotations, sweep the score threshold and identify two operating points: the stringent cutoff where 10% of negative (non-BGC) domains are false positives, and the lenient cutoff where 80% of positive (in-cluster) domains are true positives. Apply each threshold to binarize domain calls, then convert thresholded domains into discrete BGC regions both with and without the post-processing (region-merging) step, since merging materially changes BGC-level precision. Match predicted BGC regions against the manual annotations to compute BGC-level precision per tool and per threshold condition, writing the 4-condition comparison (DeepBGC vs ClusterFinder x 10% FPR vs 80% TPR) to bgc_precision_comparison.csv. The rationale is that the lenient threshold separates the tools: at 10% FPR precision is comparable (34% DeepBGC vs 26% ClusterFinder), but at 80% TPR DeepBGC reaches 44% precision versus 9% for ClusterFinder (>4-fold higher).

## Related tools

- **DeepBGC** (Produces per-Pfam-domain BiLSTM detection scores that are thresholded at the calibrated operating points) — https://github.com/Merck/deepbgc
- **ClusterFinder** (HMM baseline whose domain-level predictions are thresholded at the same operating points for the precision comparison)
- **HMMER** (Runs hmmscan to annotate Pfam domains that feed the domain-level scores)
- **Prodigal** (Predicts protein-coding genes upstream of Pfam annotation and domain scoring) — https://github.com/hyattpd/Prodigal/releases
- **bgc-pipeline** (Development and evaluation code used to reproduce the paper's calibration and precision-comparison analyses) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc download && deepbgc pipeline genome.fna
```

## Evaluation signals

- Reproduced BGC-level precision at the 10% FPR threshold matches ~34% (DeepBGC) vs ~26% (ClusterFinder)
- Reproduced BGC-level precision at the 80% TPR threshold matches ~44% (DeepBGC) vs ~9% (ClusterFinder), i.e. >4-fold difference
- Each threshold, when applied to labeled calibration domains, yields approximately the target rate (10% false positives or 80% true positives)
- bgc_precision_comparison.csv contains all 4 conditions (2 tools x 2 thresholds) with precision values in [0,1]
- With-vs-without post-processing comparisons are reported for each condition to quantify the effect of region merging

## Limitations

- BGC-level precision is highly dependent on the chosen operating point: ClusterFinder drops to 9% precision at the lenient 80% TPR threshold, so single-threshold comparisons can misrepresent relative performance
- Training data bias: DeepBGC was trained on existing BGC databases heavily biased towards Streptomyces, which can affect score calibration on poorly characterized bacteria
- Threshold-dependence is not fully resolved: some well-studied BGC classes are recovered regardless of threshold while others are more threshold-dependent

## Evidence

- [results] Under the stringent 10% FPR cutoff on the manually annotated genomes, BGC-level precision was comparable between DeepBGC and ClusterFinder (34% vs 26%): [evidence span withheld — anchor status: not_found]
- [results] ClusterFinder's lenient-threshold predictions fell to 9% precision while DeepBGC reached 44% (>4-fold higher): [evidence span withheld — anchor status: not_found]
