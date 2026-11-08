---
name: minhash-sketching-of-gene-clusters
description: Use when a metagenomic/metatranscriptomic profiling pipeline receives
  multiple predicted gene clusters (antiSMASH/gutSMASH GenBank output dirs) that must
  be deduplicated to avoid ambiguous read mapping downstream.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/SRA accessions) — that belongs to the bigmap-download
  module, not the redundancy-filtering step
- The goal is abundance/expression quantification or differential analysis of already-filtered
  representatives — use bigmap-map and bigmap-analyse instead
- Input gene clusters are not in antiSMASH/gutSMASH output directory format (the module
  expects their GenBank output folders, produced beforehand)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: MASH
  role: MinHash-sketches extracted gene-cluster protein sequences to compute fast
    pairwise similarity for the first redundancy-filtering round
- name: BiG-MAP
  role: Hosts the bigmap-family module that performs the sketching, clustering, and
    medoid selection
  repo: https://github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Optional second redundancy-filtering round that groups the representative
    gene clusters into GCFs more accurately
- name: gutSMASH
  role: Source of predicted metabolic gene cluster GenBank output folders that form
    the module's input
  repo: https://github.com/victoriapascal/gutsmash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:c055d093694679132e00296a4126bd2ccdb3e1b48bc8b5a275aeaa218c1c9e83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/medema-group/BiG-MAP
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - MASH
  - gutSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# MinHash sketching of gene clusters

## Summary

Redundancy-filtering of antiSMASH/gutSMASH-predicted gene clusters by sketching their protein sequences with MASH MinHash, clustering at a similarity cutoff, and selecting the medoid as each cluster's nonredundant representative before metagenomic read mapping.

## When to use

Use when a metagenomic/metatranscriptomic profiling pipeline receives multiple predicted gene clusters (antiSMASH/gutSMASH GenBank output dirs) that must be deduplicated to avoid ambiguous read mapping downstream.

## When NOT to use

- Input is raw sequencing data (FASTQ/SRA accessions) — that belongs to the bigmap-download module, not the redundancy-filtering step
- The goal is abundance/expression quantification or differential analysis of already-filtered representatives — use bigmap-map and bigmap-analyse instead
- Input gene clusters are not in antiSMASH/gutSMASH output directory format (the module expects their GenBank output folders, produced beforehand)

## Inputs

- Output directories of one or more antiSMASH or gutSMASH runs (GenBank files of predicted gene clusters)
- Similarity threshold -tg for gene clusters (fraction 0–1, default 0.8)
- Similarity threshold -th for housekeeping genes (fraction 0–1, default 0.1)
- Optional: processed Pfam-A.hmm database (uncompressed and hmmpressed) for the BiG-SCAPE filtering round

## Outputs

- BiG-MAP.GCF.fna — reference file of representative gene-cluster sequences to map WGS reads to
- BiG-MAP.GCF.bed — bedfile to extract core regions in bigmap-map
- BiG-MAP.GCs.json — dictionary containing the MASH-based gene cluster families
- BiG-MAP.GCF.json — dictionary containing the BiG-SCAPE GCFs
- Nonredundant representative gene-cluster collection (one medoid per similarity cluster, optionally with GCF assignments)

## How to apply

Extract the protein sequences from the predicted gene clusters in the input antiSMASH or gutSMASH GenBank output directories, then sketch them with MASH MinHash to estimate pairwise sequence similarity cheaply. Cluster the sketches with the -tg similarity threshold (default 0.8; a value of 0 makes every gene cluster its own family, 1 merges all into one family) and pick the medoid — the member with minimal cumulative distance — as each group's representative so downstream mapping has one sequence per redundant group. Optionally run a second redundancy-filtering round with BiG-SCAPE, for which a processed Pfam-A.hmm (downloaded from InterPro and hmmpressed with HMMER) is required; the README strongly recommends this second step for more accurate filtering. The related -th threshold (default 0.1) governs housekeeping-gene similarity and yields HGF representatives processed alongside the GCFs. Write the nonredundant representative collection plus GCF assignments as the module's output files.

## Related tools

- **MASH** (MinHash-sketches extracted gene-cluster protein sequences to compute fast pairwise similarity for the first redundancy-filtering round)
- **BiG-MAP** (Hosts the bigmap-family module that performs the sketching, clustering, and medoid selection) — https://github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Optional second redundancy-filtering round that groups the representative gene clusters into GCFs more accurately)
- **gutSMASH** (Source of predicted metabolic gene cluster GenBank output folders that form the module's input) — https://github.com/victoriapascal/gutsmash

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- The four output files exist: BiG-MAP.GCF.fna, BiG-MAP.GCF.bed, BiG-MAP.GCs.json, and BiG-MAP.GCF.json
- The nonredundant collection contains exactly one representative (medoid) per MASH similarity cluster formed at the -tg cutoff (0.8 by default)
- Setting -tg to 0 yields one family per gene cluster; setting it to 1 yields one large family containing all gene clusters — the output must track the threshold accordingly
- BiG-SCAPE GCF assignments in BiG-MAP.GCF.json cover only the representative clusters that survived MASH filtering

## Limitations

- MASH-based filtering at the default 0.8 similarity cutoff is coarse; the README recommends the optional BiG-SCAPE round for more accurate redundancy filtering
- BiG-SCAPE filtering requires installing BiG-SCAPE separately (conda install bigscape) plus a current hmmpressed Pfam-A database
- The antiSMASH/gutSMASH runs producing the input folders must be executed beforehand; the module does not predict gene clusters itself
- BiG-MAP has been tested only on Ubuntu

## Evidence

- [intro] MinHash-based redundancy filtering avoids ambiguous mapping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] MASH is the first redundancy-filtering step with a default 0.8 cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] bigmap-family takes anti-/gutSMASH output directories as input: [evidence span withheld — anchor status: external_doc]
- [readme] -tg threshold semantics and default: [evidence span withheld — anchor status: external_doc]
- [readme] Second BiG-SCAPE filtering round and its Pfam prerequisite: [evidence span withheld — anchor status: external_doc]
