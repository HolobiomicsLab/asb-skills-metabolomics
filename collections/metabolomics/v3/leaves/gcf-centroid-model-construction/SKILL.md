---
name: gcf-centroid-model-construction
description: Use when you have antiSMASH-predicted BGC GenBank files for hundreds
  of thousands of genomes and need scalable, non-pairwise GCF models (instead of O(n²)
  pairwise clustering) that can later map new or fragmented BGCs.
when_to_use_negative:
- The goal is exhaustive all-vs-all BGC similarity networks rather than GCF models
  — pairwise tools like BiG-SCAPE give different, network-based output but at O(n²)
  cost
- The BGC set is tiny or already a curated reference set (e.g. MIBiG entries only),
  where pairwise comparison and manual inspection remain tractable
- Most input BGCs are fragmented; since --complete-only excludes them from model building,
  a dataset dominated by fragments yields weak centroid models
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: 'Primary engine: projects BGCs into Euclidean space and builds GCF centroid
    models in a non-pairwise, near-linear fashion using --complete-only and threshold
    T'
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Upstream predictor supplying the BGC GenBank files that populate the BiG-SLiCE
    input folder (run with taxon options and --minlength 5000)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:2da18339242867e69c19fef613a596c5fd60acc43b8817a44ace5c841629e76d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gcf-centroid-model-construction

## Summary

Build Gene Cluster Family (GCF) centroid models by running BiG-SLiCE's clustering phase with --complete-only, projecting BGCs into Euclidean space so that only non-fragmented BGCs define GCF models to which fragmented BGCs can later be assigned.

## When to use

Use when you have antiSMASH-predicted BGC GenBank files for hundreds of thousands of genomes and need scalable, non-pairwise GCF models (instead of O(n²) pairwise clustering) that can later map new or fragmented BGCs.

## When NOT to use

- The goal is exhaustive all-vs-all BGC similarity networks rather than GCF models — pairwise tools like BiG-SCAPE give different, network-based output but at O(n²) cost
- The BGC set is tiny or already a curated reference set (e.g. MIBiG entries only), where pairwise comparison and manual inspection remain tractable
- Most input BGCs are fragmented; since --complete-only excludes them from model building, a dataset dominated by fragments yields weak centroid models

## Inputs

- BiG-SLiCE input folder containing antiSMASH-predicted BGC GenBank files (with .clusters.txt / taxonomy metadata per the wiki input-folder spec)
- BiG-SLiCE HMM database downloaded via download_bigslice_hmmdb
- A chosen threshold parameter T (e.g. T=300 from {300, 600, 900, 1,200, 1,500})

## Outputs

- SQLite3 database in the output folder holding processed input BGCs, GCF centroid models, and cluster/membership assignments
- GCF models usable for --query mode placement of new or fragmented BGCs
- Per-BGC GCF membership values d (used with --n_ranks 1 mapping)

## How to apply

Prepare a BiG-SLiCE input folder from antiSMASH-predicted BGC GenBank files, then run 'bigslice -i <input_folder> <output_folder>' with the --complete-only parameter so that only non-fragmented BGCs (802,287 of 1,225,071, i.e. 65%, in the reference run) are used to build GCF centroid models. Choose a distance threshold T from {300, 600, 900, 1,200, 1,500}; larger T produces fewer, broader GCF models, so test multiple values if downstream α-diversity estimates are intended. BiG-SLiCE projects BGCs into Euclidean space and applies a partitional clustering algorithm with near-linear time complexity, avoiding the quadratic pairwise comparisons that made BiG-SCAPE require an estimated 37,000 hours on a 36-core CPU. Store all processed input data, GCF models, and cluster assignments in the SQLite3 database inside the output folder; fragmented BGCs are excluded from model building and are mapped afterward via --query mode with --n_ranks 1 based on membership values d. Note that BiG-SLiCE v2.0 uses cosine-like (l2-normalized) distances and pyHMMER with PFAM 35.0, which changes clustering sensitivity relative to the v1 Euclidean-distance runs in the paper.

## Related tools

- **BiG-SLiCE** (Primary engine: projects BGCs into Euclidean space and builds GCF centroid models in a non-pairwise, near-linear fashion using --complete-only and threshold T) — https://github.com/medema-group/bigslice
- **antiSMASH** (Upstream predictor supplying the BGC GenBank files that populate the BiG-SLiCE input folder (run with taxon options and --minlength 5000))

## Examples

```
bigslice -i <input_folder> <output_folder> --complete_only
```

## Evaluation signals

- The --complete-only clustering phase must use exactly the non-fragmented subset — in the reference run 802,287 (65%) of the 1,225,071 input BGCs were used to build GCF models
- GCF model count must decrease monotonically as T increases (T = {300, 600, 900, 1,200, 1,500} produces a decreasing number of GCF models; e.g. T=900 yields 29,955 GCFs)
- The output-folder SQLite3 database must be populated with all input BGCs (1,225,071 BGCs from 209,206 genomes and MAGs in the reference run) and their cluster assignments
- The full run must scale near-linearly: the reference completed in ~240 hours on a 36-core CPU server with 128 GB RAM — far below BiG-SCAPE's estimated 37,000 hours
- 'bigslice --version' must report the expected version and HMM database md5s before the run

## Limitations

- Only non-fragmented ('complete') BGCs enter model building; fragmented BGCs depend on --query membership mapping and may be assigned to coarser models
- Choice of T strongly controls GCF granularity; no single threshold is canonical — the paper tested five values and used T=300 and T=900 jointly for α-diversity estimates
- BiG-SLiCE v2.0 switched to cosine-like (l2-normalized) distances and PFAM 35.0 models, so results are not directly comparable to the v1 Euclidean-space runs reported in the paper; no pre-processed v2 result of the 1.2M-BGC run is yet available
- BGCs not in antiSMASH/MIBiG format (e.g. from ClusterFinder or DeepBGC) require conversion via the provided generate_antismash_gbk.py script before input

## Evidence

- [results] the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [results] Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models: "Five different threshold values (T = {300, 600, 900, 1,200, 1,500}) were tested, producing a decreasing number of GCF models"
- [abstract] By representing them in Euclidean space, BiG-SLiCE can group BGCs into GCFs in a non-pairwise, near-linear fashion.: "By representing them in Euclidean space, BiG-SLiCE can group BGCs into GCFs in a non-pairwise, near-linear fashion."
- [readme] bigslice -i <input_folder> <output_folder>: "user@local:~$ bigslice -i <input_folder> <output_folder>"
- [readme] Clustering now uses cosine-like (via l2-normalization) distances: "Clustering now uses __cosine-like__ (via l2-normalization) distances (as in https://www.nature.com/articles/s41564-022-01110-2)"
- [intro] even the currently fastest tool (a tool that we previously developed, BiG-SCAPE [22]) will require an estimated 37,000 hours of runtime on a 36-core CPU: "even the currently fastest tool (a tool that we previously developed, BiG-SCAPE [22]) will require an estimated 37,000 hours of runtime on a 36-core CPU"
