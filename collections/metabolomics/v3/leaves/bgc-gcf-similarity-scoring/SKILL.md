---
name: bgc-gcf-similarity-scoring
description: Use when a new antiSMASH-annotated BGC needs to be assigned to existing
  GCF models from a prior BiG-SLiCE run (stored as an SQLite3 database), and a full
  re-clustering of the corpus would be wasteful.
when_to_use_negative:
- The input BGC is not an antiSMASH output and cannot be converted with the provided
  GenBank converter script
- No prior BiG-SLiCE clustering result (SQLite3 GCF model database) exists — full
  clustering with 'bigslice -i <input_folder> <output_folder>' must be run first
- The goal is de novo discovery of entirely new GCF structures rather than placement
  into known families; query mode only scores against precomputed centroid models
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: BiG-SLiCE
  role: 'Performs the query-mode scoring: extracts domain-based features of the query
    BGC and computes membership distances against stored GCF centroid models from
    the SQLite3 database'
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Generates the antiSMASH output folder that is the required input format for
    BiG-SLiCE --query mode
- name: pyHMMER
  role: Python-native HMM scanning library used by BiG-SLiCE 2.0 to detect Pfam domains
    underlying the feature vectors
  repo: https://github.com/althonos/pyhmmer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:fce737999cb420d28508a430bb064377d993327ac52b3c2f0ade2e8dd90aa69e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH
  - pyHMMER
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# bgc-gcf-similarity-scoring

## Summary

Scores a newly sequenced BGC against precomputed gene cluster family (GCF) centroid models using BiG-SLiCE's query mode, returning a ranked list of top-N matching GCFs with membership/similarity distances to assess function and novelty without rerunning full clustering.

## When to use

Use when a new antiSMASH-annotated BGC needs to be assigned to existing GCF models from a prior BiG-SLiCE run (stored as an SQLite3 database), and a full re-clustering of the corpus would be wasteful.

## When NOT to use

- The input BGC is not an antiSMASH output and cannot be converted with the provided GenBank converter script
- No prior BiG-SLiCE clustering result (SQLite3 GCF model database) exists — full clustering with 'bigslice -i <input_folder> <output_folder>' must be run first
- The goal is de novo discovery of entirely new GCF structures rather than placement into known families; query mode only scores against precomputed centroid models

## Inputs

- antiSMASH output folder for the query BGC (required input format for --query mode)
- BiG-SLiCE output folder from a prior clustering run, containing the SQLite3 database of GCF centroid models
- Integer value for --n_ranks (default N = 3 top-scoring GCFs returned)

## Outputs

- Ranked list of the top-(n_ranks) matching GCFs, each with its similarity/membership measurement
- Ranked list of similar individual BGCs for the query
- Query results viewable in the flask-based interactive web-app output

## How to apply

Prepare the query BGC as an antiSMASH output folder, since the query input must come from antiSMASH (for non-antiSMASH BGCs, first convert GenBank + descriptor files using the provided generate_antismash_gbk.py script). Run BiG-SLiCE with the --query flag, passing the antiSMASH output folder and the output folder containing the latest precomputed clustering result (SQLite3 database of GCF centroid models), setting --n_ranks to the number of family hits to return (default N = 3). BiG-SLiCE extracts the query's domain-based features (pHMM-detected Pfam domains, scanned via pyHMMER in v2.0) and computes membership distances against the stored GCF centroid models in Euclidean feature space. Collect the ranked top-(n_ranks) GCFs with their similarity measurements to judge the query BGC's function and/or novelty, optionally inspecting results through the flask-based interactive web-app served from the output folder.

## Related tools

- **BiG-SLiCE** (Performs the query-mode scoring: extracts domain-based features of the query BGC and computes membership distances against stored GCF centroid models from the SQLite3 database) — https://github.com/medema-group/bigslice
- **antiSMASH** (Generates the antiSMASH output folder that is the required input format for BiG-SLiCE --query mode)
- **pyHMMER** (Python-native HMM scanning library used by BiG-SLiCE 2.0 to detect Pfam domains underlying the feature vectors) — https://github.com/althonos/pyhmmer

## Examples

```
bigslice --query <antismash_output_folder> --n_ranks 3 <output_folder>
```

## Evaluation signals

- The run returns a ranked list of exactly the top-(n_ranks) matching GCFs, each accompanied by a similarity measurement, as specified by --query mode
- The output folder passed to --query contains a valid SQLite3 database from a prior clustering run; query succeeds only against the latest clustering result in that folder
- Membership distance values (d) are computed and reported for each ranked GCF, consistent with the paper's membership-value mapping scheme
- Query results can be opened in the flask web-app served from the output folder (e.g. http://0.0.0.0:5000) without schema errors

## Limitations

- The query input must be an antiSMASH-generated result; BGCs from ClusterFinder, DeepBGC, or manually refined borders need prior conversion via generate_antismash_gbk.py
- Query mode scores only against GCF centroid models from a previously computed clustering result; it cannot discover families absent from that model set
- Membership distances depend on the clustering threshold T of the original run (the paper tested T = {300, 600, 900, 1200, 1500}); fragmented BGCs were excluded from model building via --complete-only, which affects what models exist to match against
- No pre-processed GCF result for BiG-SLiCE v2 was yet available; the released ~1.2M-BGC precomputed result applies to v1

## Evidence

- [abstract] BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine: "BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine"
- [results] the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [readme] Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
- [readme] Top-(n_ranks) matching GCFs will be returned along with their similarity measurements. You can then view the query results using the user interactive output: "Top-(n_ranks) matching GCFs will be returned along with their similarity measurements. You can then view the query results using the user interactive output"
- [results] the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
