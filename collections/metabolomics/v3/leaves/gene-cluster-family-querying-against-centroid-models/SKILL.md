---
name: gene-cluster-family-querying-against-centroid-models
description: Use when you have one or more new antiSMASH-annotated BGCs and an existing
  BiG-SLiCE SQLite3 result database, and need to rank-match the new BGCs against precomputed
  GCF models instead of re-clustering.
when_to_use_negative:
- When no pre-computed BiG-SLiCE GCF models exist — query mode requires a prior clustering
  run stored as an SQLite3 result database; otherwise run a full bigslice -i <input_folder>
  <output_folder> clustering analysis first.
- When the BGC of interest is not in (and cannot be converted to) antiSMASH output
  format — the query input must be an antiSMASH-generated result.
- When pairwise BGC-level network comparison at fine granularity is required (e.g.,
  BiG-SCAPE-style sequence similarity networks) rather than centroid-based family
  assignment.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0080
tools:
- name: BiG-SLiCE
  role: 'Runs the --query mode: extracts query BGC domain-based features and matches
    them against stored GCF centroid models, returning top-N ranked GCF hits with
    similarity measurements'
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Produces the antiSMASH output folder that is the required input format for
    the query BGC
- name: pyHMMER
  role: Underlying Python-native HMM scanning library used by BiG-SLiCE 2.0 for pHMM
    domain feature extraction
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:2f454e20ee4e178a924d3a130f76fd73c14d018ed6a2533140fc0cf4fadf2b9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH
  - pyHMMER
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-cluster-family-querying-against-centroid-models

## Summary

Use BiG-SLiCE's --query mode to place newly sequenced (putative) BGCs into previously computed gene cluster families (GCFs) by matching domain-based features against stored GCF centroid models, without rerunning full clustering. This lets practitioners assess the function and/or novelty of a new BGC in seconds rather than re-clustering the entire corpus.

## When to use

Use when you have one or more new antiSMASH-annotated BGCs and an existing BiG-SLiCE SQLite3 result database, and need to rank-match the new BGCs against precomputed GCF models instead of re-clustering.

## When NOT to use

- When no pre-computed BiG-SLiCE GCF models exist — query mode requires a prior clustering run stored as an SQLite3 result database; otherwise run a full bigslice -i <input_folder> <output_folder> clustering analysis first.
- When the BGC of interest is not in (and cannot be converted to) antiSMASH output format — the query input must be an antiSMASH-generated result.
- When pairwise BGC-level network comparison at fine granularity is required (e.g., BiG-SCAPE-style sequence similarity networks) rather than centroid-based family assignment.

## Inputs

- antiSMASH output folder containing the query BGC(s) of interest
- BiG-SLiCE output folder containing the latest pre-computed clustering result (SQLite3 database of GCF centroid models)
- --n_ranks integer parameter specifying how many top-scoring GCF hits to return

## Outputs

- Ranked list of the top-(n_ranks) matching GCFs with their similarity/membership measurements
- Ranked list of similar BGCs within the matching GCFs
- Query results viewable in the flask-based user interactive output web-app

## How to apply

Prepare the query BGC as an antiSMASH output folder, since the query input must be an antiSMASH-generated result (BGCs from tools like ClusterFinder, DeepBGC, or manually refined borders must first be converted using the generate_antismash_gbk.py converter script). Invoke BiG-SLiCE with the --query flag, passing the antiSMASH output folder and the output folder holding the latest pre-computed clustering result (an SQLite3 database of GCF models), setting --n_ranks to the desired number of returned family hits (the paper's workflow used N = 3 top-scoring GCFs by default). BiG-SLiCE extracts the query BGC's domain-based features and computes membership distances against the stored GCF centroid models from the previous run. Collect the ranked list of the top-(n_ranks) matching GCFs with their similarity measurements, and optionally inspect results in BiG-SLiCE's flask-based interactive web-app launched via start_server.sh. Note that clustering in BiG-SLiCE 2.0 uses cosine-like (l2-normalized) distances with PFAM 35.0 pHMM databases, so queries should be run against a result database built with the same version for consistent distance semantics.

## Related tools

- **BiG-SLiCE** (Runs the --query mode: extracts query BGC domain-based features and matches them against stored GCF centroid models, returning top-N ranked GCF hits with similarity measurements) — https://github.com/medema-group/bigslice
- **antiSMASH** (Produces the antiSMASH output folder that is the required input format for the query BGC)
- **pyHMMER** (Underlying Python-native HMM scanning library used by BiG-SLiCE 2.0 for pHMM domain feature extraction) — https://github.com/althonos/pyhmmer

## Examples

```
bigslice --query <antismash_output_folder> --n_ranks 3 <output_folder>
```

## Evaluation signals

- The run returns a ranked list of exactly the top-(n_ranks) matching GCFs, each accompanied by its similarity measurement, as specified by the --query mode.
- The output folder's SQLite3 database is updated/queried consistently and the query results are viewable in the flask web-app (http://0.0.0.0:5000).
- Query BGC features were extracted with the same pHMM database version as the pre-computed models (bigslice-models-2022-11-30, PFAM 35.0 for v2.0) so membership distances are comparable.
- BiG-SLiCE --version output confirms the expected version and HMM database md5 checksums (e.g., Biosynthetic-pfam md5: 37495cac452bf1dd8aff2c4ad92065fe) before querying.
- Biologically, the top-ranked GCF assignment should be consistent with the BGC's known/predicted function (e.g., proximity to MIBiG reference BGCs of known function).

## Limitations

- There is currently no pre-processed result for BiG-SLiCE v2; the ~1.2M BGC pre-computed result referenced in the README was generated with v1 (a 17GB zipped download), and no HMMER path configuration applies to v1.x while v2 uses pyHMMER.
- Query mode requires the input to be an antiSMASH output folder; BGCs from ClusterFinder, DeepBGC, or with manually-refined borders need conversion via the provided generate_antismash_gbk.py script.
- Membership distances are only meaningful against GCF models built with the same feature space (domain-based pHMM features) and distance metric (cosine-like in v2.0).

## Evidence

- [abstract] BiG-SLiCE provides a query mode that efficiently places newly sequenced BGCs into previously computed GCFs: "BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine"
- [readme] Query mode CLI and top-N output: "bigslice --query <antismash_output_folder> --n_ranks <int> <output_folder>"
- [readme] Ranked GCF list with similarity measurements: "Top-(n_ranks) matching GCFs will be returned along with their similarity measurements. You can then view the query results using the user interactive output"
- [readme] Query input must be antiSMASH output: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
- [results] Fragmented BGCs mapped via membership assignment: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [readme] Cosine-like distance in v2.0: "Clustering now uses __cosine-like__ (via l2-normalization) distances (as in https://www.nature.com/articles/s41564-022-01110-2)"
