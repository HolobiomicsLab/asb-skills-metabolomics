---
name: bgc-feature-extraction-from-antismash-output
description: Use when a newly sequenced putative BGC — available as an antiSMASH output
  folder — must be placed into previously computed BiG-SLiCE gene cluster families
  (GCFs) to assess its function and/or novelty without rerunning a full clustering
  analysis.
when_to_use_negative:
- No pre-computed BiG-SLiCE clustering result (SQLite3 GCF models) exists yet — a
  full clustering run on the input folder must be performed first before --query can
  be used
- The goal is de novo pairwise BGC network comparison on a small dataset, where BiG-SCAPE's
  pairwise similarity network approach is the intended tool rather than centroid-model
  matching
- The BGC of interest cannot be provided in antiSMASH output format and cannot be
  converted from GenBank + descriptor file using the provided converter script
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: BiG-SLiCE
  role: Extracts the query BGC's domain-based features and computes membership distances
    against stored GCF centroid models, returning a ranked list of matching GCFs
  repo: https://github.com/medema-group/bigslice
- name: antiSMASH
  role: Produces the antiSMASH output folder format required as the query input for
    BiG-SLiCE --query mode
  repo: https://antismash.secondarymetabolites.org/
- name: ClusterFinder
  role: Alternative BGC predictor whose output must be converted to antiSMASH-like
    GenBank format before querying
  repo: https://github.com/petercim/ClusterFinder
- name: DeepBGC
  role: Alternative BGC predictor whose output must be converted to antiSMASH-like
    GenBank format before querying
  repo: https://github.com/Merck/deepbgc
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa154
    title: 'BiG-SLiCE: A highly scalable tool maps the diversity of 1.2 million biosynthetic
      gene clusters'
  source_content_hash: sha256:062c8627c17497e780800c6e584b5208c50b5ac2923f989aa49b00e232e7f0f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/bigslice
  tools:
  - BiG-SLiCE
  - ClusterFinder
  - DeepBGC
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bgc-feature-extraction-from-antismash-output

## Summary

BiG-SLiCE's query mode extracts domain-based features (via pyHMMER pHMM scanning against PFAM models) from a newly sequenced BGC supplied as an antiSMASH output folder, then computes membership distances against precomputed GCF centroid models stored in an SQLite3 database to rank the closest gene cluster families.

## When to use

Use when a newly sequenced putative BGC — available as an antiSMASH output folder — must be placed into previously computed BiG-SLiCE gene cluster families (GCFs) to assess its function and/or novelty without rerunning a full clustering analysis.

## When NOT to use

- No pre-computed BiG-SLiCE clustering result (SQLite3 GCF models) exists yet — a full clustering run on the input folder must be performed first before --query can be used
- The goal is de novo pairwise BGC network comparison on a small dataset, where BiG-SCAPE's pairwise similarity network approach is the intended tool rather than centroid-model matching
- The BGC of interest cannot be provided in antiSMASH output format and cannot be converted from GenBank + descriptor file using the provided converter script

## Inputs

- antiSMASH-generated output folder for the query BGC (or a GenBank file + comma-separated BGC descriptor file converted via generate_antismash_gbk.py)
- BiG-SLiCE output folder containing the latest pre-computed clustering result as an SQLite3 database of GCF centroid models (e.g., the ~17GB zipped pre-processed result on ~1.2M microbial BGCs from NCBI)
- --n_ranks integer (number of top-ranked GCF hits to return)

## Outputs

- Ranked list of the top-(n_ranks) matching GCFs, each with its similarity/membership measurement
- Matching BGCs similar to the query BGC within the top-scoring GCFs
- Query results viewable through the flask-based interactive web-app or the SQLite3 output database

## How to apply

Prepare the query BGC as an antiSMASH-generated output folder (if the BGC comes from ClusterFinder, DeepBGC, or manually refined borders, first convert it using BiG-SLiCE's generate_antismash_gbk.py converter script, which takes a GenBank file plus a comma-separated BGC descriptor file). Invoke BiG-SLiCE with --query, passing the antiSMASH output folder and the output folder holding the latest pre-computed clustering result (the SQLite3 database of GCF centroid models), setting --n_ranks to the number of ranked family hits wanted (default top-N = 3; the paper used n_ranks=1 when mapping fragmented BGCs back onto GCFs via membership values d). BiG-SLiCE extracts the query BGC's domain-based features and computes its membership distance against the stored GCF centroid models in the Euclidean feature space — clustering sensitivity depends on the distance metric of the model version (v2.0 uses cosine-like distances via l2-normalization against PFAM 35.0 pHMMs). Collect the ranked top-(n_ranks) GCFs with their similarity measurements; optionally inspect results through the flask-based web-app in the output folder or query the SQLite database directly with SQL.

## Related tools

- **BiG-SLiCE** (Extracts the query BGC's domain-based features and computes membership distances against stored GCF centroid models, returning a ranked list of matching GCFs) — https://github.com/medema-group/bigslice
- **antiSMASH** (Produces the antiSMASH output folder format required as the query input for BiG-SLiCE --query mode) — https://antismash.secondarymetabolites.org/
- **ClusterFinder** (Alternative BGC predictor whose output must be converted to antiSMASH-like GenBank format before querying) — https://github.com/petercim/ClusterFinder
- **DeepBGC** (Alternative BGC predictor whose output must be converted to antiSMASH-like GenBank format before querying) — https://github.com/Merck/deepbgc

## Examples

```
bigslice --query <antismash_output_folder> --n_ranks 3 <output_folder>
```

## Evaluation signals

- The run returns a ranked list of exactly top-(n_ranks) matching GCFs, each accompanied by a similarity measurement, as specified by --query mode
- The query succeeded against the latest clustering result contained inside the supplied output folder (SQLite3 database of GCF models), not a fresh full clustering run
- Membership distances are on the same scale as the threshold parameter used to build the models (the paper tested T = {300, 600, 900, 1200, 1500})
- The query input was a valid antiSMASH output folder — runs on non-antiSMASH inputs fail or require prior conversion with generate_antismash_gbk.py
- Results can be cross-checked by opening the flask web-app (bash <output_folder>/start_server.sh) or by SQL queries against the output SQLite3 database

## Limitations

- The query input must be an antiSMASH-generated output folder; BGCs from other predictors (ClusterFinder, DeepBGC) or with manually refined borders need the GenBank converter script first
- As of BiG-SLiCE v2, there is no pre-processed GCF model result available for download (the ~1.2M-BGC pre-processed result applies to v1.x)
- Interpreting results requires SQL(ite) querying for programmatic access, which the authors note has a steeper learning curve than tabular output files
- Fragmented BGCs are handled via membership mapping against GCF models (the paper used n_ranks=1 for this), which may be less informative than full ranked matches

## Evidence

- [readme] Query mode matches BGC to precomputed GCF models: "Using the `--query` mode, you can perform a blazing-fast query of a putative BGC against the pre-processed set of Gene Cluster Family (GCF) models"
- [abstract] Query mode places new BGCs into computed GCFs: "BiG-SLiCE also provides a "query mode" that can efficiently place newly sequenced BGCs into previously computed GCFs, plus a powerful output visualization engine"
- [results] Membership mapping of BGCs onto GCFs: "the full input datasets were queried back against the GCF models to map the fragmented BGCs onto their corresponding GCFs based on the calculated membership values d"
- [readme] Query returns ranked GCF hits with similarity: "Top-(n_ranks) matching GCFs will be returned along with their similarity measurements. You can then view the query results using the user interactive output"
- [readme] Converter for non-antiSMASH BGC input: "which will take a (genome) GenBank file along with a comma-separated descriptor file for every BGCs to be generated"
