---
name: fragmented-bgc-membership-assignment
description: Use when an antiSMASH-derived BGC dataset contains fragmented clusters
  (partial BGCs on contig edges) that were excluded from GCF model building, and each
  fragment must be assigned to an existing GCF rather than left unclustered.
when_to_use_negative:
- Your dataset contains only complete, non-fragmented BGCs — build GCFs directly in
  a single clustering run instead of a separate query pass
- No prior BiG-SLiCE clustering output (GCF models) exists for the query to run against
  — --query mode requires a pre-computed result in the output folder
- You need novel de novo families for fragmented BGCs rather than placement into existing
  GCF models
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3173
tools:
- name: BiG-SLiCE
  role: Performs both the --complete-only clustering phase that builds GCF models
    and the --query mode that maps fragmented BGCs onto those models via membership
    values d
  repo: github.com/medema-group/bigslice
- name: antiSMASH 5.0
  role: Predicted the source BGCs (v5.1.1 with --minlength 5000); its output folder
    is the input format for --query mode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:c3d60c6b995f29a1cbf78010963d41d0f29b0479b191e5a8ab0c67f30b595209
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SLiCE
  - antiSMASH 5.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fragmented-bgc-membership-assignment

## Summary

Maps fragmented BGCs (e.g. those truncated at contig boundaries in antiSMASH output) onto pre-computed BiG-SLiCE gene cluster family (GCF) models by querying the full input dataset back against centroid models built only from non-fragmented BGCs, assigning each fragment a GCF via membership value d.

## When to use

Use when an antiSMASH-derived BGC dataset contains fragmented clusters (partial BGCs on contig edges) that were excluded from GCF model building, and each fragment must be assigned to an existing GCF rather than left unclustered.

## When NOT to use

- Your dataset contains only complete, non-fragmented BGCs — build GCFs directly in a single clustering run instead of a separate query pass
- No prior BiG-SLiCE clustering output (GCF models) exists for the query to run against — --query mode requires a pre-computed result in the output folder
- You need novel de novo families for fragmented BGCs rather than placement into existing GCF models

## Inputs

- antiSMASH-predicted BGC GenBank files for ~209,206 genomes and MAGs (including fragmented BGCs)
- A prior BiG-SLiCE output folder containing GCF centroid models built with --complete-only (SQLite3 database)

## Outputs

- GCF membership assignments for fragmented BGCs, each with a ranked GCF hit and membership value d (n_ranks=1 gives the single best GCF)
- Updated SQLite3 database in the output folder containing all 1,225,071 BGCs from 209,206 genomes mapped to GCFs
- Queryable result viewable via the output folder's Flask-based interactive web app

## How to apply

First run the clustering phase ('bigslice -i <input_folder> <output_folder>') with the --complete-only parameter so only the 802,287 (65%) non-fragmented BGCs build the GCF centroid models at a chosen threshold T (the paper tested T = {300, 600, 900, 1,200, 1,500}). Then run BiG-SLiCE in --query mode ('bigslice --query <antismash_output_folder> --n_ranks <int> <output_folder>') with --n_ranks 1, which queries the full input dataset, including fragmented BGCs, back against the stored GCF models and returns the top-ranked matching GCF with its membership value d. The rationale is that fragments are too truncated to serve as reliable centroids but can still be assigned by distance to GCF centroids in Euclidean feature space. All processed input data, GCF models, and cluster assignments are persisted in the SQLite3 database inside the output folder. Avoid --complete-only in the query phase, since the goal is to place every input BGC, not only complete ones.

## Related tools

- **BiG-SLiCE** (Performs both the --complete-only clustering phase that builds GCF models and the --query mode that maps fragmented BGCs onto those models via membership values d) — github.com/medema-group/bigslice
- **antiSMASH 5.0** (Predicted the source BGCs (v5.1.1 with --minlength 5000); its output folder is the input format for --query mode)

## Examples

```
bigslice --query <antismash_output_folder> --n_ranks 1 <output_folder>
```

## Evaluation signals

- The query phase with --n_ranks 1 returns exactly one top-ranked GCF per queried BGC with a similarity/membership measurement
- The final populated datastore contains exactly 1,225,071 BGCs from 209,206 genomes and MAGs (i.e. all fragmented BGCs were assigned, not dropped)
- Only 802,287 (65%) non-fragmented BGCs were used to build the GCF centroid models during the --complete-only clustering phase
- Assignments are persisted in the SQLite3 database inside the output folder and are retrievable via SQL queries
- Query results can be inspected in the interactive Flask web app served from the output folder

## Limitations

- Membership values depend on the threshold T chosen when building GCF models; the paper tested T = {300, 600, 900, 1,200, 1,500}, producing decreasing numbers of GCF models
- The pre-processed result on ~1.2M microbial BGCs available for download was produced with BiG-SLiCE v1; the README notes there is currently no pre-processed result for BiG-SLiCE v2
- Fragmented BGCs are assigned by distance to GCF centroids, so a highly truncated fragment may be misassigned to its nearest rather than true family

## Evidence

- [results] Mapping fragmented BGCs onto GCFs via membership assignment with best-hit basis (n_ranks=1): "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [results] Excluding fragmented BGCs from model building using --complete-only: "the “–complete-only” parameter was used for the clustering phase, using 802,287 (65%) non-fragmented BGCs from the input data to build the GCF models"
- [readme] Query mode places new BGCs into pre-computed GCFs: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
