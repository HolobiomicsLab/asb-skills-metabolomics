---
name: gene-cluster-family-grouping
description: Use when SMASH-predicted gene clusters (antiSMASH or gutSMASH GenBank
  output folders for a genome collection) contain redundant clusters that must be
  dereplicated and grouped into GCFs before metagenomic/metatranscriptomic read mapping.
when_to_use_negative:
- The gene clusters are already dereplicated with a single representative per similarity
  group, so re-clustering would only add noise
- The input is raw metagenomic/metatranscriptomic reads or a finished abundance table
  rather than SMASH genome-mining output folders — the correct modules are bigmap-map
  or bigmap-analyse
- No prior antiSMASH/gutSMASH run has been performed on the genomes; this module does
  not predict gene clusters itself
edam_operation: http://edamontology.org/operation_3432
tools:
- name: BiG-MAP
  role: Hosts the bigmap-family module that performs MASH-based redundancy filtering,
    medoid selection, and HGF identification
  repo: github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: Second redundancy-filtering step that groups representative gene clusters
    into GCFs at a 0.3 distance cutoff, using a hmmpressed Pfam-A database
  repo: https://github.com/medema-group/BiG-SCAPE/wiki
- name: MASH
  role: MinHash sketching of extracted protein sequences for fast pairwise similarity
    estimation at the 0.8 default cutoff
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:ed6cddbf1132145282412532e5668079742cb84c1a83d1e14de66dc711fe8225
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - MASH
  - bigmap-analyse
  - bigmap-family
  - bigmap-map
  - gutSMASH
  - ncbi/sra-tools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# gene-cluster-family-grouping

## Summary

Builds a nonredundant representative gene-cluster collection from antiSMASH/gutSMASH output by MinHash (MASH) sketching of cluster protein sequences at a 0.8 similarity cutoff, selecting medoid representatives, and optionally grouping representatives into gene cluster families (GCFs) with BiG-SCAPE. This prevents ambiguous read mapping in the downstream BiG-MAP.map step.

## When to use

Use when SMASH-predicted gene clusters (antiSMASH or gutSMASH GenBank output folders for a genome collection) contain redundant clusters that must be dereplicated and grouped into GCFs before metagenomic/metatranscriptomic read mapping.

## When NOT to use

- The gene clusters are already dereplicated with a single representative per similarity group, so re-clustering would only add noise
- The input is raw metagenomic/metatranscriptomic reads or a finished abundance table rather than SMASH genome-mining output folders — the correct modules are bigmap-map or bigmap-analyse
- No prior antiSMASH/gutSMASH run has been performed on the genomes; this module does not predict gene clusters itself

## Inputs

- Output directories of antiSMASH or gutSMASH runs (per-genome predicted gene-cluster GenBank files)
- Pfam-A.hmm downloaded from InterPro and processed with hmmpress (required for the BiG-SCAPE second filtering round)

## Outputs

- BiG-MAP.GCF.fna — reference FASTA of representative gene-cluster sequences for read mapping
- BiG-MAP.GCF.bed — BED file to extract core regions in bigmap-map
- BiG-MAP.GCs.json — dictionary of MASH-based gene cluster groupings
- BiG-MAP.GCF.json — dictionary of BiG-SCAPE gene cluster families

## How to apply

Load the output directories of antiSMASH or gutSMASH runs and extract protein sequences from all predicted metabolic gene clusters (MGCs). Sketch the proteins with MASH and cluster at the default 0.8 similarity threshold (tunable via -tg between 0, where every cluster is its own family, and 1, where all clusters form one family); for each redundant group select the medoid — the member with minimal cumulative distance — as the nonredundant representative, and separately cluster housekeeping genes at a 0.1 threshold (-th) to define housekeeping gene families (HGFs). Optionally run a second redundancy-filtering round by grouping representatives into GCFs with BiG-SCAPE (distance cutoff 0.3), which requires the latest Pfam-A.hmm processed with hmmpress. Write the four module outputs (BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, BiG-MAP.GCF.json) for use by bigmap-map.

## Related tools

- **BiG-MAP** (Hosts the bigmap-family module that performs MASH-based redundancy filtering, medoid selection, and HGF identification) — github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Second redundancy-filtering step that groups representative gene clusters into GCFs at a 0.3 distance cutoff, using a hmmpressed Pfam-A database) — https://github.com/medema-group/BiG-SCAPE/wiki
- **MASH** (MinHash sketching of extracted protein sequences for fast pairwise similarity estimation at the 0.8 default cutoff)

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- All four output artifacts exist after the run: BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json
- The output collection contains exactly one nonredundant representative per MASH similarity cluster, and each representative is the medoid (minimal cumulative distance) of its group
- Varying -tg behaves as documented: a threshold of 0 makes every gene cluster its own family, a threshold of 1 merges all clusters into one large family
- Downstream bigmap-map produces unambiguous per-sample abundance profiles, confirming no redundant clusters remain in BiG-MAP.GCF.fna
- In reproduced case studies (e.g., 1,520 CGR genomes or 1,440 HMOD oral genomes), the GCF grouping supports the reported counts of differentially abundant clusters under Kruskal-Wallis testing

## Limitations

- The module does not predict gene clusters — it requires prior antiSMASH or gutSMASH output directories as input
- BiG-SCAPE-based GCF grouping is optional and strongly recommended by the authors for more accurate redundancy filtering, but it adds the dependency on a current, hmmpressed Pfam-A.hmm from InterPro
- The similarity threshold -tg is a sensitive decision point: at 0 every cluster stands alone and at 1 all clusters collapse into one family, so the default 0.8 must be validated per dataset
- BiG-MAP has only been tested on Ubuntu, and the optional filterBAM dependency applies only to unpaired ancient read data

## Evidence

- [intro] MinHash-based redundancy filtering of gene clusters to avoid ambiguous mapping: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] MASH default 0.8 similarity cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] Module input requirements: [evidence span withheld — anchor status: external_doc]
- [readme] Module outputs: [evidence span withheld — anchor status: external_doc]
