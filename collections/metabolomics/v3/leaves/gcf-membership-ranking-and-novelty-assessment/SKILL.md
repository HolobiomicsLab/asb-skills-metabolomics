---
name: gcf-membership-ranking-and-novelty-assessment
description: Use when one or a few new antiSMASH-annotated BGCs must be assigned to
  existing GCF models quickly, or when judging whether a BGC is novel versus a member
  of known biosynthetic families.
when_to_use_negative:
- When no pre-computed BiG-SLiCE GCF model database exists — this mode queries prior
  clustering results and cannot replace an initial clustering run (bigslice -i <input_folder>
  <output_folder>).
- When the BGC of interest is not in antiSMASH output format and no conversion via
  the generate_antismash_gbk.py script has been performed.
- When a full de novo re-clustering of a large corpus is the goal (e.g., building
  new GCF models with --complete-only and threshold T); query mode only assigns to
  existing models.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SLiCE
  role: 'Performs the query-mode GCF assignment: extracts domain-based features of
    the query BGC and ranks matches against pre-computed GCF centroid models in the
    SQLite3 output database'
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Generates the antiSMASH output folder format required as the query-mode input;
    BGCs from other predictors must be converted into this format
  repo: https://antismash.secondarymetabolites.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:c0775383fb8403dfb01db9e17e55099b69725c9a61c2e0d4fb1d49a53e5b9ba0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# GCF membership ranking and novelty assessment via BiG-SLiCE query mode

## Summary

Places newly sequenced (putative) BGCs into previously computed BiG-SLiCE gene cluster family (GCF) centroid models without rerunning full clustering, returning a ranked top-N list of matching GCFs with membership scores to assess the BGC's function and/or novelty.

## When to use

Use when one or a few new antiSMASH-annotated BGCs must be assigned to existing GCF models quickly, or when judging whether a BGC is novel versus a member of known biosynthetic families.

## When NOT to use

- When no pre-computed BiG-SLiCE GCF model database exists — this mode queries prior clustering results and cannot replace an initial clustering run (bigslice -i <input_folder> <output_folder>).
- When the BGC of interest is not in antiSMASH output format and no conversion via the generate_antismash_gbk.py script has been performed.
- When a full de novo re-clustering of a large corpus is the goal (e.g., building new GCF models with --complete-only and threshold T); query mode only assigns to existing models.

## Inputs

- antiSMASH output folder containing the query BGC(s)
- BiG-SLiCE output folder with the latest pre-computed clustering result (SQLite3 GCF model database)
- --n_ranks integer (number of top GCF hits to return)

## Outputs

- Ranked list of top-(n_ranks) matching GCFs with membership/similarity measurements for each query BGC
- Ranked list of similar BGCs to the query, usable to determine function and/or novelty
- Query results viewable through the flask-based interactive output web-app

## How to apply

Prepare the query BGC as an antiSMASH output folder, since BiG-SLiCE's query mode requires antiSMASH-generated input; BGCs from other predictors (ClusterFinder, DeepBGC) or manually refined borders must first be converted via the provided generate_antismash_gbk.py converter script. Run BiG-SLiCE with the --query flag, passing the antiSMASH output folder and the output folder holding the latest pre-computed clustering result (the SQLite3 database of GCF models), setting --n_ranks to the number of family hits to return (default top-3 per the paper's task description). BiG-SLiCE extracts the query BGC's domain-based features and computes membership distances against the stored GCF centroid models, then returns the ranked top-(n_ranks) matching GCFs with their similarity measurements. Optionally inspect results in the flask-based interactive web-app served from the output folder.

## Related tools

- **BiG-SLiCE** (Performs the query-mode GCF assignment: extracts domain-based features of the query BGC and ranks matches against pre-computed GCF centroid models in the SQLite3 output database) — https://github.com/medema-group/bigslice
- **antiSMASH** (Generates the antiSMASH output folder format required as the query-mode input; BGCs from other predictors must be converted into this format) — https://antismash.secondarymetabolites.org/

## Examples

```
bigslice --query <antismash_output_folder> --n_ranks <int> <output_folder>
```

## Evaluation signals

- Run returns a ranked list of the top-(n_ranks) matching GCFs, each accompanied by its similarity measurement, as specified by --query mode.
- Membership distances are computed against GCF centroid models stored in the SQLite3 database from the prior clustering run — confirm the queried output folder is the latest clustering result.
- Query BGC input must be a valid antiSMASH output folder; failure to prepare this format means the skill was not applied correctly.
- Results can be inspected in the flask-based web-app from the output folder (pip install -r requirements.txt; bash start_server.sh) — a successful server launch confirms output integrity.

## Limitations

- The query input must be an antiSMASH-generated result; BGCs from ClusterFinder, DeepBGC, or with manually-refined borders require conversion via the provided GenBank converter script first.
- There is currently no pre-processed result (pre-computed GCF model database) for BiG-SLiCE v2; the ~1.2M BGC NCBI result available for download was generated with v1, so the model database must be built locally.
- Query mode assigns to previously computed GCFs only; it cannot discover families absent from the reference clustering run (novelty is inferred by the absence/score of close GCF matches).

## Evidence

- [abstract] BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine: "BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine"
- [readme] Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
- [readme] Top-(n_ranks) matching GCFs will be returned along with their similarity measurements. You can then view the query results using the user interactive output: "Top-(n_ranks) matching GCFs will be returned along with their similarity measurements. You can then view the query results using the user interactive output"
- [results] the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [readme] BiG-SLiCE's output folder contains both the processed input data (in the form of an SQLite3 database file) and some scripts that power a mini web-app to visualize that data: [evidence span withheld — anchor status: not_found]
