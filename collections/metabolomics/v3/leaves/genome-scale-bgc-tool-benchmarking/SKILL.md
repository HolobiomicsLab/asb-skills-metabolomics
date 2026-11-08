---
name: genome-scale-bgc-tool-benchmarking
description: Use when comparing BGC prediction tool performance (counts and prediction-set
  overlap) across a large genome collection, e.g. reproducing DeepBGC 10926 vs antiSMASH
  13865 vs ClusterFinder 62491 on 3376 RefSeq genomes.
when_to_use_negative:
- Only a single genome or a handful of genomes is available — the benchmark relies
  on genome-scale counts (3376 genomes) for statistically meaningful tool comparison
- The goal is to train or fine-tune a BGC detector rather than compare existing tool
  outputs — use deepbgc train / deepbgc prepare instead
- Genomes are metagenomic assemblies rather than RefSeq bacterial reference genomes
  — ClusterFinder/antiSMASH comparison was established on reference genomes
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0080
tools:
- name: DeepBGC
  role: Primary BGC detection tool run via 'deepbgc pipeline' with strict 2% FPR detector
    cutoff on all 3376 genomes
  repo: https://github.com/Merck/deepbgc
- name: antiSMASH
  role: Rule-based BGC prediction baseline run with default settings for count and
    overlap comparison
- name: ClusterFinder
  role: HMM-based BGC prediction baseline run with default lenient settings (predicted
    62491 BGCs, most permissive of the three)
- name: Prodigal
  role: Automatic gene calling within the DeepBGC pipeline when genes are not already
    annotated
  repo: https://github.com/hyattpd/Prodigal/releases
- name: HMMER
  role: Automatic Pfam domain detection (hmmscan) within the DeepBGC pipeline to produce
    Pfam TSV inputs for the detector
- name: bgc-pipeline (DVC)
  role: Repository holding the data and DVC-managed commands used to develop DeepBGC
    and evaluate it against ClusterFinder and antiSMASH
  repo: https://github.com/Merck/bgc-pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz654
    title: A deep learning genome-mining strategy for biosynthetic gene cluster prediction
  source_content_hash: sha256:30483382c500f560077a9160d875a7ad4f23d3ec9543fd1269b2dc717aa15594
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
  - bgc-pipeline (DVC)
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# genome-scale-bgc-tool-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark biosynthetic gene cluster (BGC) prediction tools (DeepBGC, antiSMASH, ClusterFinder) across thousands of RefSeq bacterial genomes by aggregating per-genome BGC counts and computing pairwise overlap of predicted BGC sequences, as done in Figure 5A of the DeepBGC paper.

## When to use

Use when comparing BGC prediction tool performance (counts and prediction-set overlap) across a large genome collection, e.g. reproducing DeepBGC 10926 vs antiSMASH 13865 vs ClusterFinder 62491 on 3376 RefSeq genomes.

## When NOT to use

- Only a single genome or a handful of genomes is available — the benchmark relies on genome-scale counts (3376 genomes) for statistically meaningful tool comparison
- The goal is to train or fine-tune a BGC detector rather than compare existing tool outputs — use deepbgc train / deepbgc prepare instead
- Genomes are metagenomic assemblies rather than RefSeq bacterial reference genomes — ClusterFinder/antiSMASH comparison was established on reference genomes

## Inputs

- 3376 RefSeq bacterial genome sequence files
- DeepBGC trained detector models and Pfam database (via deepbgc download)
- antiSMASH default rule-based settings configuration
- ClusterFinder default lenient HMM settings

## Outputs

- Per-genome BGC counts for DeepBGC, antiSMASH, and ClusterFinder
- Pairwise overlap fractions of predicted BGC sequences between tools
- Summary comparison table (figure5A_bgc_overlap.csv)

## How to apply

Download the 3376 RefSeq bacterial genome sequences and fetch trained detector models plus the Pfam database with 'deepbgc download'. Run 'deepbgc pipeline' on each genome (Prodigal gene calling and HMMER hmmscan Pfam domain detection run automatically), applying the strict detector domain-score cutoff corresponding to a 2% false positive rate. Run antiSMASH on the same genomes with default rule-based settings and ClusterFinder with its default lenient HMM settings. Aggregate per-genome BGC counts per tool and compute the fraction of predicted BGC sequence shared between each pair of tools (DeepBGC overlapped ~90% with ClusterFinder, ~39% with antiSMASH, ~5% unique). Write the comparison table (e.g. figure5A_bgc_overlap.csv) and validate against Figure 5A values.

## Related tools

- **DeepBGC** (Primary BGC detection tool run via 'deepbgc pipeline' with strict 2% FPR detector cutoff on all 3376 genomes) — https://github.com/Merck/deepbgc
- **antiSMASH** (Rule-based BGC prediction baseline run with default settings for count and overlap comparison)
- **ClusterFinder** (HMM-based BGC prediction baseline run with default lenient settings (predicted 62491 BGCs, most permissive of the three))
- **Prodigal** (Automatic gene calling within the DeepBGC pipeline when genes are not already annotated) — https://github.com/hyattpd/Prodigal/releases
- **HMMER** (Automatic Pfam domain detection (hmmscan) within the DeepBGC pipeline to produce Pfam TSV inputs for the detector)
- **bgc-pipeline (DVC)** (Repository holding the data and DVC-managed commands used to develop DeepBGC and evaluate it against ClusterFinder and antiSMASH) — https://github.com/Merck/bgc-pipeline

## Examples

```
deepbgc download && deepbgc pipeline genome.fa
```

## Evaluation signals

- Per-tool BGC counts match Figure 5A: DeepBGC = 10926, antiSMASH = 13865, ClusterFinder = 62491 across 3376 genomes
- ClusterFinder predicts >4.5x more BGCs than antiSMASH and >5.5x more than DeepBGC
- ~90% of DeepBGC predictions overlap ClusterFinder; ~39% overlap antiSMASH (35% overlap both); ~5% (566 BGCs) are unique to DeepBGC
- Summary table figure5A_bgc_overlap.csv reproduces the reported pairwise overlap fractions
- DeepBGC pipeline logs confirm the strict detector domain-score cutoff corresponding to 2% FPR was applied

## Limitations

- ClusterFinder's default lenient settings inflate BGC counts (62491), so raw count comparisons reflect permissiveness, not accuracy
- DeepBGC's models were trained on existing BGC databases heavily biased towards BGCs from natural product 'workhorses' such as Streptomyces, limiting novel BGC detection in poorly characterized bacteria
- Some BGC classes were mis-identified in the S. coelicolor reference genome example, and some classes' recovery was threshold-dependent

## Evidence

- [readme] This repository provides data and examples that were used for development of DeepBGC and its evaluation with ClusterFinder and antiSMASH: [evidence span withheld — anchor status: external_doc]
