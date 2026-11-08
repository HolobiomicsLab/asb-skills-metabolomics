---
name: secondary-metabolite-novelty-assessment
description: Use when a BiG-SLiCE output database (SQLite3) of clustered GCFs is available
  and you must rank GCFs/taxa by novelty, or when a new antiSMASH BGC must be placed
  against precomputed GCF models to judge novelty.
when_to_use_negative:
- When the dataset is small enough that pairwise comparison tools like BiG-SCAPE can
  run in reasonable time — the Euclidean-space GCF pipeline is aimed at the massive
  datasets (~1.2M BGCs) where pairwise methods are infeasible
- When no BiG-SLiCE clustering run (and its SQLite3 output) exists for the BGCs of
  interest — the skill operates on precomputed GCF centroid features, not raw genomes
- When the goal is functional annotation of a BGC already matching a known MIBiG family
  closely — novelty assessment adds no information
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: Provides the Euclidean-space GCF centroid features, the SQLite3 output database,
    and the --query mode used to measure novelty of GCF models or place new BGCs against
    precomputed GCFs
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Supplies the predicted BGCs (with class definitions) that constitute the input
    cluster set being scored for novelty
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:a3329778b493d98dd1d10d680fa46c79c895f21ba57d62f107948239516f0f73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# secondary-metabolite-novelty-assessment

## Summary

Quantifies how far predicted BGCs or BiG-SLiCE gene cluster family (GCF) models lie from experimentally verified reference BGCs (MIBiG v2.0), flagging 'uncharted biosynthetic potential' in a dataset or taxon.

## When to use

Use when a BiG-SLiCE output database (SQLite3) of clustered GCFs is available and you must rank GCFs/taxa by novelty, or when a new antiSMASH BGC must be placed against precomputed GCF models to judge novelty.

## When NOT to use

- When the dataset is small enough that pairwise comparison tools like BiG-SCAPE can run in reasonable time — the Euclidean-space GCF pipeline is aimed at the massive datasets (~1.2M BGCs) where pairwise methods are infeasible
- When no BiG-SLiCE clustering run (and its SQLite3 output) exists for the BGCs of interest — the skill operates on precomputed GCF centroid features, not raw genomes
- When the goal is functional annotation of a BGC already matching a known MIBiG family closely — novelty assessment adds no information

## Inputs

- BiG-SLiCE output SQLite3 database with BGC, class, GCF, and genome taxonomy tables
- MIBiG v2.0 reference BGC feature vectors (1,910 experimentally verified entries)
- antiSMASH output folder for a putative BGC (for query mode)

## Outputs

- Histogram of nearest-MIBiG distances for every GCF model
- Per-taxon novelty flags identifying GCF models far from MIBiG reference features (uncharted biosynthetic potential)
- Ranked list of top-n_ranks matching GCFs and BGCs with similarity measurements (query mode)

## How to apply

Load the BiG-SLiCE output SQLite3 database and select the GCF models from a chosen threshold run (the paper used the 29,955 GCFs from T=900). Take each GCF's centroid feature vector in Euclidean space and compute its distance to the closest MIBiG v2.0 reference BGC features (the reference set comprises all 1,910 MIBiG v2.0 entries), then plot the distribution of these distances as a histogram — GCFs with large nearest-MIBiG distances encode biosynthetic chemistry not represented among verified clusters. Optionally combine distance-based novelty with taxonomic aggregation: taxa whose GCF models consistently lie far from MIBiG features (e.g., the distinct fungal clades a1–a4) mark uncharted potential. For individual new BGCs, use BiG-SLiCE's --query mode with --n_ranks to retrieve a ranked list of matching GCFs and BGCs with similarity measurements instead of recomputing distances. Note that v1 used Euclidean distances whereas BiG-SLiCE v2.0 switched to cosine-like (l2-normalized) distances, so distance values are not comparable across versions.

## Related tools

- **BiG-SLiCE** (Provides the Euclidean-space GCF centroid features, the SQLite3 output database, and the --query mode used to measure novelty of GCF models or place new BGCs against precomputed GCFs) — https://github.com/medema-group/bigslice
- **antiSMASH** (Supplies the predicted BGCs (with class definitions) that constitute the input cluster set being scored for novelty)

## Examples

```
bigslice --query <antismash_output_folder> --n_ranks <int> <output_folder>
```

## Evaluation signals

- At the paper's settings, the T=900 clustering yields 29,955 GCF models whose distances to the closest MIBiG BGC features can be computed — reproducing the analysis should recover this GCF count
- The MIBiG reference set must contain all 1,910 MIBiG v2.0 entries; a smaller/older reference changes all distance values
- The distance histogram should be non-negative with GCFs matching known compound classes (e.g., NRP/PK) at low distances and taxon-specific clades (fungal a1–a4) enriched at high distances
- Query-mode output must return exactly top-n_ranks GCFs with similarity measurements per queried BGC
- Aggregated BGC counts reconciled against the paper's totals of 1,225,071 BGCs from 209,206 genomes and MAGs

## Limitations

- Novelty is defined only relative to MIBiG v2.0 reference BGCs — GCFs far from MIBiG may still match unreported but real natural products
- The pre-processed 1.2M-BGC result (full_run_result.zip) was produced with BiG-SLiCE v1; there is currently no pre-processed result for v2, and v2's cosine-like distance metric is not comparable to v1 Euclidean distances
- GCF models were built with --complete-only (802,287, i.e., 65%, non-fragmented BGCs), so fragmented BGCs are only assigned post hoc via membership values and do not influence model centroids

## Evidence

- [results] Distance of every GCF model to its closest MIBiG BGC: "we took the 29,955 GCFs calculated from T = 900, measured the distance of every GCF model against their closest MIBiG BGC features"
- [results] MIBiG v2.0 as verified reference set: "we incorporated all 1,910 entries from MIBiG v2.0 [78] as a reference set of known and experimentally verified BGCs"
- [abstract] Query mode for novelty determination: "BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine"
- [readme] Readme: ranked query output helps determine novelty: "You will get a ranked list of GCFs and BGCs similar to the BGC in question, which will help in determining the function and/or novelty of said BGC."
- [readme] Readme: no v2 pre-processed result yet: "there is currently no pre-processed result for BiG-SLiCE v2, we will work to make it available soon."
