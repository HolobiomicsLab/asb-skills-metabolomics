---
name: gene-cluster-redundancy-clustering
description: Use when you have antiSMASH/gutSMASH output directories for many genomes
  and need a nonredundant gene-cluster reference set (and optional GCF assignments)
  before mapping metagenomic/metatranscriptomic reads.
when_to_use_negative:
- The gene-cluster collection is already nonredundant or already carries GCF assignments
  — re-clustering would not change the mapping reference.
- The goal is to quantify abundance/expression from metagenomic or metatranscriptomic
  reads — that is the bigmap-map step, which consumes this module's outputs.
- Only a single genome with one predicted gene cluster is available — there is no
  redundancy to filter and clustering is meaningless.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BiG-MAP
  role: bigmap-family module performs the redundancy filtering (MASH) and GCF grouping
    (BiG-SCAPE) and emits the mapping reference files
  repo: github.com/medema-group/BiG-MAP
- name: MASH
  role: MinHash-sketches extracted protein sequences to compute fast pairwise similarity
    and perform the first redundancy-filtering round at the 0.8 cutoff
- name: BiG-SCAPE
  role: second redundancy-filtering round that groups representative gene clusters
    into GCFs at a 0.3 distance cutoff
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: gutSMASH
  role: upstream predictor whose output directories (predicted MGCs in GenBank) are
    the direct input to bigmap-family
  repo: github.com/victoriapascal/gutsmash
- name: antiSMASH
  role: alternative upstream BGC predictor whose output directories are the direct
    input to bigmap-family
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:e01b92e43d045d586268e454719b10b2a6efdbc161f076fc61c91e30133749c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - MASH
  - antiSMASH
  - gutSMASH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# gene-cluster-redundancy-clustering

## Summary

Redundancy-filtering and clustering of predicted metabolic gene clusters (MGCs/BGCs) into a nonredundant representative collection, as implemented in the BiG-MAP.family module: MASH MinHash sketching at a 0.8 similarity cutoff for deduplication, medoid selection, and optional BiG-SCAPE grouping into gene cluster families (GCFs).

## When to use

Use when you have antiSMASH/gutSMASH output directories for many genomes and need a nonredundant gene-cluster reference set (and optional GCF assignments) before mapping metagenomic/metatranscriptomic reads.

## When NOT to use

- The gene-cluster collection is already nonredundant or already carries GCF assignments — re-clustering would not change the mapping reference.
- The goal is to quantify abundance/expression from metagenomic or metatranscriptomic reads — that is the bigmap-map step, which consumes this module's outputs.
- Only a single genome with one predicted gene cluster is available — there is no redundancy to filter and clustering is meaningless.

## Inputs

- Output directories of antiSMASH or gutSMASH runs (predicted gene clusters in GenBank format) for multiple genomes
- Processed Pfam-A.hmm database (Pfam-A.hmm.gz from Interpro, uncompressed and hmmpress-ed) if the BiG-SCAPE second filtering round is used

## Outputs

- BiG-MAP.GCF.bed — bedfile to extract core regions in bigmap-map
- BiG-MAP.GCF.fna — reference file to map the WGS reads to
- BiG-MAP.GCs.json — dictionary that contains the (MASH-derived) gene-cluster groups
- BiG-MAP.GCF.json — dictionary that contains the BiG-SCAPE gene cluster families (GCFs)

## How to apply

Run bigmap-family on the output directories of anti- or gutSMASH runs: it extracts the protein sequences of all predicted gene clusters from the GenBank files, sketches them with MASH MinHash sketches, and clusters them at a 0.8 similarity threshold (flag -tg) so that highly similar gene clusters do not cause ambiguous read mapping downstream. For each similarity cluster the medoid (minimal cumulative distance) is selected as the nonredundant representative. Optionally, a second redundancy-filtering round groups representatives into GCFs with BiG-SCAPE at a 0.3 distance cutoff, which the README strongly recommends for more accurate filtering; BiG-SCAPE requires a processed Pfam-A.hmm database (downloaded from Interpro, uncompressed, and hmmpress-ed). The optional -tg parameter controls the similarity threshold: 0 puts every gene cluster in its own family, 1 merges all gene clusters into one family. Housekeeping gene similarity uses a separate default threshold of 0.1 (-th).

## Related tools

- **BiG-MAP** (bigmap-family module performs the redundancy filtering (MASH) and GCF grouping (BiG-SCAPE) and emits the mapping reference files) — github.com/medema-group/BiG-MAP
- **MASH** (MinHash-sketches extracted protein sequences to compute fast pairwise similarity and perform the first redundancy-filtering round at the 0.8 cutoff)
- **BiG-SCAPE** (second redundancy-filtering round that groups representative gene clusters into GCFs at a 0.3 distance cutoff) — https://github.com/medema-group/BiG-SCAPE/wiki
- **gutSMASH** (upstream predictor whose output directories (predicted MGCs in GenBank) are the direct input to bigmap-family) — github.com/victoriapascal/gutsmash
- **antiSMASH** (alternative upstream BGC predictor whose output directories are the direct input to bigmap-family)

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- The family module output file exists and contains exactly one nonredundant representative per MASH similarity cluster, matching expected medoid (minimal cumulative distance) selection.
- All four expected output files are produced: BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json.
- With the default -tg 0.8 threshold, redundant gene clusters (protein-sequence similarity above 0.8) collapse into single representatives; setting -tg 0 must yield one family per gene cluster and -tg 1 one large family.
- Every input predicted gene cluster is accounted for in exactly one similarity group / GCF in the JSON dictionaries (no orphan clusters).
- If BiG-SCAPE is enabled, representatives are grouped at the 0.3 distance cutoff and BiG-MAP.GCF.json reflects those family assignments.

## Limitations

- BiG-MAP has only been tested on Ubuntu; other platforms are unverified.
- The BiG-SCAPE second filtering round is optional, so skipping it leaves only the coarser MASH 0.8 protein-similarity deduplication, which the authors consider less accurate.
- BiG-SCAPE requires the latest processed Pfam-A.hmm database (downloaded, uncompressed, hmmpress-ed); missing or stale Pfam files break this round.
- Thresholds are user-tunable: -tg 0 or -tg 1 degenerate cases (each cluster its own family vs. one large family) show the clustering is sensitive to the cutoff.

## Evidence

- [intro] MinHash-based redundancy filtering of gene clusters to avoid ambiguous mapping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] MASH first filtering round with 0.8 default cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] BiG-SCAPE second round strongly recommended: [evidence span withheld — anchor status: external_doc]
- [readme] Input is anti-/gutSMASH output directories: [evidence span withheld — anchor status: external_doc]
- [readme] Family module outputs consumed by downstream mapping: "BiG-MAP.GCF.bed = Bedfile to extract core regions in bigmap-map
BiG-MAP.GCF.fna = Reference file to map the WGS reads to"
